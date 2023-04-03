# React学习心得



## State 如何设计

- 可以先把需要的都设置为State
- 接着把可以通过props / state推导的去掉
- 考虑如果 state 经常一起更新的 可以合并 如果不会同时true的 可以设置为string 如 typing submitting
- 还可以考虑合并一些state 如果成为了对象或者数组 记得考虑要浅拷贝的问题(可以通过useImmer去解决)



##  preserving-and-resetting-state 什么时候会更新/什么时候会保留状态

- 位置相同的时候**position in the UI tree** 例如使用三元表达式 不会更新

``` react
{isPlayerA ? (
        <Counter person="Taylor" />
      ) : (
        <Counter person="Sarah" />
      )}
```

- 位置不同的时候 如

``` react	
{isPlayerA &&
        <Counter person="Taylor" />
      }
      {!isPlayerA &&
        <Counter person="Sarah" />
      }
```



## useRef

| refs                                                         | state                                                        |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `useRef(initialValue)` returns `{ current: initialValue }`   | `useState(initialValue)` returns the current value of a state variable and a state setter function ( `[value, setValue]`) |
| Doesn’t trigger re-render when you change it.                | Triggers re-render when you change it.                       |
| Mutable—you can modify and update `current`’s value outside of the rendering process. | “Immutable”—you must use the state setting function to modify state variables to queue a re-render. |
| You shouldn’t read (or write) the `current` value during rendering. | You can read state at any time. However, each render has its own [snapshot](https://react.dev/learn/state-as-a-snapshot) of state which does not change. |

 #### 父子节点

- forwardRef
- useImperativeHandle



### useEffect

- 调用两次是为了检查错误找bug，建议在开发环境不要关闭，生产环境关闭
- 根据依赖项可以实现 didMount update unmount
- Effect也有缺点 例如 无法在服务端跑 会造成网络瀑布