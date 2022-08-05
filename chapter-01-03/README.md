## 前言

实际项目中编译开发主要分成了 `开发模式`和`生产模式`两种甚至更多种情况，本篇主要讲述`rollup.js`怎么配置`开发模式`。主要要素有一下三点：

- 1.本地开发的 HTTP 服务
- 2.生成开发调试的 sourceMap 文件
- 3.不能混淆，保证编译后代码的可读性

## 实现例子

- 编译 ES6+代码
- 编译成`umd`格式(通用模块定义)
- 编译生成 sourceMap 文件
- 启动 HTTP 服务

### 步骤一 安装

```sh
  npm init -y

  ## 安装 rollup.js 基础模块
  npm i --save-dev rollup
  ## 安装 rollup.js 编译ES6+的 babel 模块
  npm i --save-dev @rollup/plugin-babel @babel/core @babel/preset-env
  ## 安装 rollup.js 编译本地开发服务插件
  npm i --save-dev rollup-plugin-serve
```

- 修改 package.json 文件 scripts

```
  "scripts": {
    "dev": "node_modules/.bin/rollup -w -c ./build/rollup.config.js"
  },
```

- `rollup` 模块是 rollup.js 编译的核心模块
- `@rollup/plugin-babel` 模块是 rollup.js 支持 babel 官方编译插件模块
- `@babel/core` 是官方 babel 编译核心模块
- `@babel/preset-env` 是官方 babel 编译解析 ES6+ 语言的扩展模块

## 步骤二 rollup 配置

- 创建 `./build/rollup.config.js`

```js
const path = require("path");
const { babel } = require("@rollup/plugin-babel");
const serve = require("rollup-plugin-serve");

const resolveFile = function (filePath) {
  return path.join(__dirname, "..", filePath);
};

module.exports = {
  input: resolveFile("src/index.js"),
  output: {
    file: resolveFile("dist/index.js"),
    format: "umd",
    sourcemap: true,
  },
  plugins: [
    babel({
      presets: ["@babel/preset-env"],
    }),
    serve({
      port: 3001,
      contentBase: [resolveFile("example"), resolveFile("dist")],
    }),
  ],
};
```

## 步骤三 创建待编译 ES6 源码

- 源码路径 `./src/index.js`

```js
import demo from "./lib/demo";

const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
console.log([...arr1, ...arr2]);
async function initDemo() {
  let data = await demo();
  console.log(data);
}
initDemo();
```

- 源码路径 `./src/lib/demo.js`

```js
function demo() {
  return new Promise((resolve, reject) => {
    try {
      setTimeout(() => {
        const obj1 = { a: 1 };
        const obj2 = { b: 2 };
        const result = { ...obj1, ...obj2 };
        resolve(result);
      }, 1000);
    } catch (err) {
      reject(err);
    }
  });
}
export default demo;
```

### 步骤 四: 编译结果

- 在项目目录下执行 `npm run dev`
- 编译结果在目录 `./dist/` 下
- 编译结果分成
  - ES5 代码文件 `./dist/index.js`
  - 源码的 sourceMap 文件 `./dist/index.js.map`
- 插件服务启动了`3001` 端口

### 步骤 五: 浏览器查看结果

- example 目录`./example/index.html`
- example 源码

```html
<html>
  <head>
    <meta charset="utf-8" />
    <script src="https://cdn.bootcss.com/babel-polyfill/6.26.0/polyfill.js"></script>
  </head>
  <body>
    <p>打开控制台看 console.log 数据</p>
    <script src="./index.js"></script>
  </body>
</html>
```

- 访问 [http://127.0.0.1:3001](http://127.0.0.1:3001)
- 打开工作台 console 就会显示可运行结果

```sh
[1, 2, 3, 4, 5, 6]
{a: 1, b: 2}
>
```
