# Babel学习笔记



### Babel的三个阶段

1. Parse 阶段 把源代码转变成 AST语法树（其实就是一个object）
2. traverse阶段 通过对AST语法树进行增删改查来进行修改
3. generator阶段 将处理过的AST语法树进行转回code 还可以生成sourceMap



##### Parse阶段

会将字面量、标识符、表达式、语句、模块语法、class 语法转成对应的 AST

可以通过[AST explorer](https://astexplorer.net/) 网站去看到生成的 AST 语法树



##### Traverse阶段

parse 出的 AST 由 @babel/traverse 来遍历和修改，babel traverse 包提供了 traverse 方法：

```js
function traverse(parent, opts)
```

![img](file:///Users/huanghualian/Library/Mobile%20Documents/com~apple~CloudDocs/%E5%89%8D%E7%AB%AF%E5%AD%A6%E4%B9%A0%E8%B5%84%E6%96%99/Babel%20%E6%8F%92%E4%BB%B6%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/4.Babel%20%E7%9A%84%20API_files/5768a7c151914586ab2a5b09b698b4d7_tplv-k3u1fbpfcp-zoom-in-crop-mark_3024_0_0_0.awebp)



###### 实操1

```js
// console输出的时候带上行列信息
import { parse } from '@babel/parser';
import _traverse from '@babel/traverse';
import types from '@babel/types';
import _generate from "@babel/generator";
const traverse = _traverse.default
const generate = _generate.default
const sourceCode = `
    console.log(1);

    function func() {
        console.info(2);
    }

    export default class Clazz {
        say() {
            console.debug(3);
        }
        render() {
            return <div>{console.error(4)}</div>
        }
    }
`;

const ast = parse(sourceCode, {
    sourceType: 'unambiguous',
    plugins: ['jsx']
});

traverse(ast, {
    CallExpression (path, state) {
        if ( types.isMemberExpression(path.node.callee) 
            && path.node.callee.object.name === 'console' 
            && ['log', 'info', 'error', 'debug'].includes(path.node.callee.property.name) 
           ) {
            const { line, column } = path.node.loc.start;
            path.node.arguments.unshift(types.stringLiteral(`filename: (${line}, ${column})`))
        }
    }
});

const { code, map } = generate(ast, { sourceMaps: true })
console.log("🚀 ~ file: index.js:54 ~ code:", code)

```

###### 实操2

```js
// console打印的时候前面插入console打印行列
import { parse } from '@babel/parser';
import _traverse from '@babel/traverse';
import types from '@babel/types';
import _generate from "@babel/generator";
import _template from '@babel/template' 
const traverse = _traverse.default
const generate = _generate.default
const template = _template.default
const sourceCode = `
    console.log(1);

    function func() {
        console.info(2);
    }

    export default class Clazz {
        say() {
            console.debug(3);
        }
        render() {
            return <div>{console.error(4)}</div>
        }
    }
`;

const ast = parse(sourceCode, {
    sourceType: 'unambiguous',
    plugins: ['jsx']
});

const targetCalleeName = ['log','info','error','debug'].map(item => `console.${item}`)

traverse(ast, {
    CallExpression(path, state) {
        if (path.node.isNew) {
            return;
        }
        const calleeName = generate(path.node.callee).code;
         if (targetCalleeName.includes(calleeName)) {
            const { line, column } = path.node.loc.start;

            const newNode = template.expression(`console.log("filename: (${line}, ${column})")`)();
            newNode.isNew = true;

            if (path.findParent(path => path.isJSXElement())) {
                path.replaceWith(types.arrayExpression([newNode, path.node]))
                path.skip();
            } else {
                path.insertBefore(newNode);
            }
        }
    }
});

const { code, map } = generate(ast, { sourceMaps: true })
console.log("🚀 ~ file: index.js:54 ~ code:", code)

```





##### genertor阶段

generate 是把 AST 打印成字符串，是一个从根节点递归打印的过程，对不同的 AST 节点做不同的处理，在这个过程中把抽象语法树中省略掉的一些分隔符重新加回来。

![img](file:///Users/huanghualian/information/%E5%89%8D%E7%AB%AF%E5%AD%A6%E4%B9%A0%E8%B5%84%E6%96%99/Babel%20%E6%8F%92%E4%BB%B6%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/8.Generator%20%E5%92%8C%20SourceMap%20%E7%9A%84%E5%A5%A5%E7%A7%98_files/04d9befc0ad54eb2822d3fb086a50cd7_tplv-k3u1fbpfcp-zoom-in-crop-mark_3024_0_0_0.awebp)



### Babel中SourceMap

[Sourmap原理分析 & vlq](http://www.qiutianaimeili.com/html/page/2019/05/89jrubx1soc.html)



### Babel中的插件和preset

##### plugin

可以将通用的逻辑封装成插件，上传到 npm 仓库来复用。

##### preset

plugin 是单个转换功能的实现，当 plugin 比较多或者 plugin 的 options 比较多的时候就会导致使用成本升高。这时候可以封装成一个 preset，用户可以通过 preset 来批量引入 plugin 并进行一些配置。preset 就是对 babel 配置的一层封装。

plugin 和 preset 是有顺序的，先 plugin 再 preset，plugin 从左到右，preset 从右到左。



### Babel 内置功能

plugin 分为了 transform、proposal、syntax 三种, preset 就是插件的集合。

插件之间的可复用的 AST 操作逻辑，需要注入的公共代码都在 helper 里。

除了注入到 AST 外，还有一部分是从 runtime 包引入的。runtime 包分为 helper、regenerator、core-js 3部分。后面两个都是社区的实现。



### Babe能做什么

1. 自动国际化 in18 (通过找literal)
2. 自动埋点 (通过找到需要埋点的token去插入AST埋点节点)
3. linter 
4. 类型检查 (通过Ts类型声明以及变量type比较实现)
5. 压缩混淆 (通过scope中rename去实现)
6. JS解释器 进行代码转译 例如 Taro、Uniapp跨多端框架 (通过AST语法树去进行调整编译)
7. 模块遍历分析 例如打包 (通过分析Import Export 以及递归查询去实现)