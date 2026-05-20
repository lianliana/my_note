# SSE 流式渲染封装实践

> 源码：`/Users/huanghualian/yuanli/tutor-student-common/packages/student-chat-utils/src/`
> 实际使用：`/Users/huanghualian/yuanli/tutor-box-ai-explanation/src/context/ClassRoomServiceContext.tsx`

---

## 一、为什么选择 SSE

AI 对话场景需要将服务端生成的内容实时推送到客户端，常见方案有三种：

| 方案 | 连接方式 | 方向 | 适用场景 |
|------|---------|------|---------|
| HTTP 轮询 | 短连接，反复请求 | 客户端主动拉 | 低频更新，实现简单 |
| WebSocket | 长连接，双向通道 | 双向 | 实时双向通信（IM、游戏） |
| SSE | 长连接，单向推送 | 服务端推客户端 | 服务端单向流式输出（AI 对话、日志流） |

SSE 在 AI 对话场景的优势：

- **协议简单**：基于普通 HTTP，无需握手升级，天然支持负载均衡和代理
- **自动重连**：浏览器原生支持断线重连，无需手动实现
- **单向足够**：AI 生成内容只需服务端推送，不需要双向通道，WebSocket 的双向能力在此场景是冗余的
- **文本协议**：格式为 `key:value\n\n`，调试直观，Nginx/CDN 可直接透传

---

## 二、整体架构

该实现参考了 Ant Design X 1.5.0 的 API 设计，但**不直接依赖 antdx**，主要差异：

- 移除 UI 组件，仅保留通信和消息管理逻辑
- 增加 App 内 JS Bridge 通道（antdx 只有 Fetch）
- App 内需要手动解析 SSE（客户端限制，无法走 Web Streams）

```
应用层 (useXChat Hook)      — 消息状态管理、生命周期
    ↓
代理层 (XAgent Class)       — 请求去重、回调防护
    ↓
请求层 (XRequest Class)     — SSE/JSON 路由、环境检测
    ↓
传输层 (xFetch + XStream)   — 字节流解析
    ↓
网络 / JS Bridge
```

---

## 三、SSE 协议解析（x-stream/index.ts）

浏览器环境下用 Web Streams API 搭三段流水线：

```
ReadableStream<Uint8Array>
    → TextDecoderStream        (字节 → 字符串)
    → splitStream()            (字符串 → 完整事件块)
    → splitPart()              (事件块 → { data, event, id, retry })
```

**splitStream**：维护 buffer，每次收到 chunk 拼进去，按 `\n\n` 切割。切出的完整块入队，最后不完整的部分留在 buffer 等下次拼接，流结束时 `flush` 把残留 buffer 送出。

**splitPart**：收到完整事件块后按 `\n` 拆行，每行用 `indexOf(':')` 定位分隔符（而非 `split`，因为 value 里可以含冒号），key 为空的行（注释行）跳过，其余组装成 `{ key: value }` 对象。

最终流支持 `for await...of` 异步迭代，每次 yield 一个 `SSEOutput` 对象。

---

## 四、消息生命周期（use-x-chat/index.ts）

消息状态：`'local' | 'loading' | 'success' | 'error'`

```
onRequest() 调用
    ↓
① createMessage(userInput, 'local')        → 追加用户消息
② createMessage(placeholder, 'loading')   → 追加占位消息（loadingMsgId 记录）

    ↓ agent.request 发起请求

onUpdate(chunk) 每次收到 chunk：
    → 第一次：删掉 loading 占位，创建新消息，状态为 'loading'（updatingMsgId 记录）
    → 后续：找到 updatingMsgId 的消息，原地更新内容

onSuccess(chunks)：
    → 找到 updatingMsgId，状态改为 'success'

onError(error)：
    → 有 requestFallback：删掉 loadingMsgId 和 updatingMsgId，追加 fallback 消息，状态为 'error'
    → 无 fallback：直接删掉这两条
```

两个 id 追踪不同的对象：
- `loadingMsgId`：最初的占位气泡
- `updatingMsgId`：流式更新的目标气泡（第一个 chunk 到来时替换占位气泡）

---

## 五、useSyncState — 解决 React 闭包陷阱

**问题**：`useState` 的值在当前渲染周期内是常量，`setState` 之后当前闭包拿到的仍是旧值。

在 `useXChat` 的异步回调链中，这是真实的问题：

```typescript
setMessages((ori) => [...ori, userMsg, loadingMsg])

// 紧接着在 agent 的异步回调里需要读最新值：
let msg = getMessages().find((info) => info.id === updatingMsgId)
// 若使用 useState，这里永远是闭包旧值，找不到刚创建的消息
```

**根本原因**：`useState` 的值是每次渲染的快照，闭包捕获的是快照副本；`useRef` 捕获的是对象引用，`.current` 可以随时修改。

**解法**：把渲染用的值和逻辑用的值分开存储。

```typescript
export default function useSyncState<T>(defaultValue) {
    const [, forceUpdate] = React.useState(0)
    const stateRef = React.useRef<T>(...)

    const setState = (action) => {
        stateRef.current = newValue    // ① 同步写入 ref（立刻生效）
        forceUpdate(prev => prev + 1)  // ② 触发重渲染（异步）
    }

    const getState = () => stateRef.current  // 始终返回最新值

    return [stateRef.current, setState, getState]
}
```

`getState()` 读的是 ref，不受渲染周期约束，可在任意异步回调中拿到最新状态。

---

## 六、实际使用示例（ClassRoomServiceContext.tsx）

### 基本用法

```typescript
// 1. 创建 agent，指定接口地址
const [agent] = useXAgent<MessageType>({
    baseURL: `${CHAT_URL}?_productId=...&version=...`,
})

// 2. 配置 useXChat
const { onRequest, messages, setMessages } = useXChat<MessageType>({
    agent,

    // 出错时展示的降级消息
    requestFallback: (_, { error }) => ({
        role: 'assistant',
        content: error.name === 'AbortError' ? 'Request is aborted' : 'Request failed...',
    }),

    // 每个 SSE chunk 到来时，将内容累积到消息上
    transformMessage: ({ originMessage, chunk }) => {
        const message = JSON.parse(chunk.data)
        return {
            id: message.id,
            role: 'assistant',
            content: `${originMessage?.content || ''}${message.choices[0].message.content}`,
        }
    },

    // 拿到 AbortController，用于主动中断请求
    resolveAbortController: controller => {
        abortController.current = controller
    },
})

// 3. 发送消息
onRequest({ chatId, message: { role: 'user', content: '用户输入' } })
```

### 将 SSE 流转为 Promise

`useXChat` 本身没有"等待完成"的能力。项目中通过 `Deferred` 将流桥接为 Promise，让上层逻辑可以 `await` 整个对话过程：

```
SSE 流（持续 chunk）
    ↓
transformMessage 处理每个 chunk
    ↓
某 chunk 中 chatStatus === ChatEnd
    ↓
sendMessageDeffer.resolve()   ← 流的结束信号转为 Promise resolve
    ↓
await sendMessageDeffer.promise   ← 上层 submit() 在此等待
```

有三个出口，均汇聚到同一个 `sendMessageDeffer.promise`：

```
chatStatus=ChatEnd → resolve()   // 正常结束
requestFallback    → reject()    // SSE 断连 / 解析错误
onTimeout (23s)    → reject()    // 超时
```

上层 `submitWithRetry` 统一 `catch` 后决定是否重试（最多 3 次）。

---

## 七、关键文件

| 功能 | 路径 |
|------|------|
| SSE 协议解析 | `student-chat-utils/src/x-stream/index.ts` |
| 请求层（SSE/JSON 路由） | `student-chat-utils/src/x-request/index.ts` |
| 代理封装 | `student-chat-utils/src/use-x-agent/index.ts` |
| 消息管理 Hook | `student-chat-utils/src/use-x-chat/index.ts` |
| 同步状态工具 | `student-chat-utils/src/use-x-chat/useSyncState.ts` |
| 性能监控 | `student-chat-utils/src/metrics/SSEPerfMetrics.ts` |
| 实际业务使用 | `tutor-box-ai-explanation/src/context/ClassRoomServiceContext.tsx` |
