# Babel学习笔记



### Babel的三个阶段

1. Parse 阶段 把源代码转变成 AST语法树（其实就是一个object）
2. traverse阶段 通过对AST语法树进行增删改查来进行修改
3. generator阶段 将处理过的AST语法树进行转回code 还可以生成sourceMap



### Parse阶段

会将字面量、标识符、表达式、语句、模块语法、class 语法转成对应的 AST

可以通过[AST explorer](https://astexplorer.net/) 网站去看到生成的 AST 语法树



### Traverse阶段

parse 出的 AST 由 @babel/traverse 来遍历和修改，babel traverse 包提供了 traverse 方法：

```js
function traverse(parent, opts)
```

![img](file:///Users/huanghualian/Library/Mobile%20Documents/com~apple~CloudDocs/%E5%89%8D%E7%AB%AF%E5%AD%A6%E4%B9%A0%E8%B5%84%E6%96%99/Babel%20%E6%8F%92%E4%BB%B6%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/4.Babel%20%E7%9A%84%20API_files/5768a7c151914586ab2a5b09b698b4d7_tplv-k3u1fbpfcp-zoom-in-crop-mark_3024_0_0_0.awebp)



### genertor阶段

generate 是把 AST 打印成字符串，是一个从根节点递归打印的过程，对不同的 AST 节点做不同的处理，在这个过程中把抽象语法树中省略掉的一些分隔符重新加回来。

![img](file:///Users/huanghualian/information/%E5%89%8D%E7%AB%AF%E5%AD%A6%E4%B9%A0%E8%B5%84%E6%96%99/Babel%20%E6%8F%92%E4%BB%B6%E9%80%9A%E5%85%B3%E7%A7%98%E7%B1%8D/8.Generator%20%E5%92%8C%20SourceMap%20%E7%9A%84%E5%A5%A5%E7%A7%98_files/04d9befc0ad54eb2822d3fb086a50cd7_tplv-k3u1fbpfcp-zoom-in-crop-mark_3024_0_0_0.awebp)



### Babel中SourceMap

[Sourmap原理分析 & vlq](http://www.qiutianaimeili.com/html/page/2019/05/89jrubx1soc.html)