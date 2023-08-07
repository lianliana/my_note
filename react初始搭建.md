## ESlint
1. 安装eslint

```js
npm i eslint -D

yarn add eslint -D

pnpm add -D eslint

```
然后在当前目录下初始化eslint配置

```// 通过npx执行eslint
npx eslint -init

// 也可以通过下面命令
node ./node_modules/.bin/eslint --init

```

2. 配置.eslintrc.js

```js
module.exports = {
  env: {
    browser: true,
    commonjs: true,
    es2021: true
  },
  extends: [
    'plugin:react/recommended',
    'standard'
  ],
  parserOptions: {
    ecmaFeatures: {
      jsx: true
    },
    ecmaVersion: 'latest'
  },
  plugins: [
    'react'
  ],
  rules: {
  }
}

```

3. 配置.eslintignore


```dist
package-lock.json
node_modules
commitlint.config.js
```

## Preitter

## Husky
1. 安装
husky是生成git的的各个hook钩子的。生成的钩子会在执行git的各个操作的时候触发。
选一种方式安装即可。我安装的是`8.0.1`版本
```js
npm install husky -D

yarn add husky -D

pnpm add -D husky 

```
2. 配置
```js
// npm7及以上可以执行
npm set-script prepare "husky install"

// npm7以下可以执行
npx npe scripts.prepare "husky install"

```
这个命令会在**package.json**的**scripts**字段里加一条 `{ "prepare": "husky install" }`

或者你也可以手动添加

然后你执行这条命令`npm run prepare`

就会在当前目录下生成一个`.husky`目录。

然后继续执行下面命令，会生成一个hook钩子，叫`pre-commit`.


```js
npx husky add .husky/pre-commit "yarn run lint"
```
有时候不想对所有文件执行eslint，只想对暂存的文件做操作，那么就可以用`lint-staged`这个依赖。

它是只对暂存区的git文件才会执行操作。其它区的不做处理。

安装，我装的是`9.5.0`版本


```js
npm install lint-staged -D

yarn add lint-staged -D

pnpm add -D lint-staged 

```
我就在`lint`的命令执行`lint-staged`，然后在`lint-staged`字段根据对匹配的文件进行代码格式化和`git add`，重新把它添加到暂存区。
配置如下：

```js
复制代码
{
  "scripts": {
    "prepare": "husky install",
    "lint": "lint-staged"
  },
  "lint-staged": {
    "*.{js,jsx}": [
      "eslint --fix",
      "git add"
    ]
  }
}
```


## commitLint
```
npm i -D @commitlint/{config-conventional,cli} 
```
然后在项目根目录新建一个 **commitlint.config.js** 文件
```js
// commitlint.config.js
module.exports = { extends: ['@commitlint/config-conventional'] };
```
上一节已经在项目中引入了 Husky，修改一下 **commit-msg** 脚本，在每次 git commit 的时候执行 commitlint 校验
```
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit $1'
# or
yarn husky add .husky/commit-msg 'yarn commitlint --edit $1'
```


## 其他
1. 注意版本兼容问题，可以尝试去换package.json中的版本，删掉node_modules和package.lock.json
2. eslint、prettier等可以通过vscode底部去看执行的日志 是否失败等
3. 可以去看对应官方文档去配置，看一些网络博客记得要看下日期，有些很老的技术博客上的配置方法已经不兼容或者不适用，则需要用新的方法去配置
4. 注意看输出的提示！！