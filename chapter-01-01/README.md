### 步骤一 安装

```sh
  npm init -y
  npm install rollup @rollup/plugin-buble --D
```
- 修改package.json 文件 scripts
> "scripts": {
>   "build": "node_modules/.bin/rollup -c ./build/rollup.config.js"
> },

- `rollup` 模块是 rollup 编译的核心模块
- `@rollup/plugin-buble` 模块是 rollup 的 ES6 编译插件
  - 功能和`babel`类似，是简化版的`babel`
  - 由于是简化版，编译速度比`babel`快一些

## 步骤二 rollup 配置

- 创建 `./build/rollup.config.js`

```js
const path = require("path");
const buble = require("@rollup/plugin-buble");

const resolve = function (filePath) {
  return path.join(__dirname, "..", filePath);
};

module.exports = {
  input: resolve("src/index.js"),
  output: {
    file: resolve("dist/index.js"),
    format: "iife",
  },
  plugins: [buble()],
};
```

## 步骤三 创建待编译 ES6 源码

- 源码路径 `./src/index.js`

```js
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const result = [...arr1, ...arr2];
console.log(result);
```

### 步骤 4: 编译结果

- 在项目目录下执行 `npm run build`
- 编译结果在目录 `./dist/` 下
- 编译成 ES5 结果为

```js
(function () {
  "use strict";

  var arr1 = [1, 2, 3];
  var arr2 = [4, 5, 6];

  var result = arr1.concat(arr2);
  console.log(result);
})();
```
