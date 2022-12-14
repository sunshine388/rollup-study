# node 模块引用

## 前言

`rollup.js`编译源码中的模块引用默认只支持 `ES6+`的模块方式`import/export`。然而大量的`npm`模块是基于`CommonJS`模块方式，这就导致了大量 `npm` 模块不能直接编译使用。所以辅助`rollup.js`编译支持 `npm`模块和`CommonJS`模块方式的插件就应运而生。

- `@rollup/plugin-commonjs` 支持`CommonJS`的模块引用的`rollup.js`插件
- `@rollup/plugin-node-resolve` 支持`npm`模块引用的`rollup.js`插件

## 实现例子

- 编译 jquery npm 模块的引用和使用

demo 例子

[https://github.com/chenshenhai/rollupjs-note/blob/master/demo/chapter-02-04/](https://github.com/chenshenhai/rollupjs-note/blob/master/demo/chapter-02-04/)

```sh
npm i

## 开发模式
npm run dev

## 生产模式
npm run build
```

## 实现步骤

### 步骤 1: 目录和准备

```sh
.
├── build ## 编译脚本慕课
│   ├── rollup.config.dev.js
│   ├── rollup.config.js
│   └── rollup.config.prod.js
├── dist ## 编译结果
│   ├── index.js
│   └── index.js.map
├── example ## 例子
│   └── index.html
├── package.json
└── src ## 编译源文件
    └── index.js
```

安装对应编译的 npm 模块

```sh
## 安装待引用的jQuery模块
npm i --save jquery

## 安装 rollup.js 基础模块
npm i --save-dev rollup

## 安装 rollup.js 编译本地开发服务插件
npm i --save-dev rollup-plugin-serve

## 安装 rollup.js 编译代码混淆插件
npm i --save-dev rollup-plugin-uglify

## 安装 rollup.js 编译ES6+的 babel 模块
npm i --save-dev @rollup/plugin-babel @babel/core @babel/preset-env

## 加载node模块依赖和CommonJS格式的插件
npm i --save-dev @rollup/plugin-commonjs @rollup/plugin-node-resolve
```

### 步骤 2: rollup 配置

- 编译基本配置 `./build/rollup.config.js`

```js
const path = require("path");
const { babel } = require("@rollup/plugin-babel");
const nodeResolve = require("@rollup/plugin-node-resolve");
const commonjs = require("@rollup/plugin-commonjs");

const resolveFile = function (filePath) {
  return path.join(__dirname, "..", filePath);
};

const babelOptions = {
  presets: ["@babel/preset-env"],
};

module.exports = [
  {
    input: resolveFile("src/index.js"),
    output: {
      file: resolveFile("dist/index.js"),
      format: "umd",
    },
    plugins: [nodeResolve(), commonjs(), babel(babelOptions)],
  },
];
```

- `开发模式`配置基本 `./build/rollup.config.dev.js`

```js
const path = require("path");
const serve = require("rollup-plugin-serve");
const configList = require("./rollup.config");

const resolveFile = function (filePath) {
  return path.join(__dirname, "..", filePath);
};
const PORT = 3000;

const devSite = `http://127.0.0.1:${PORT}`;
const devPath = path.join("example", "index.html");
const devUrl = `${devSite}/${devPath}`;

setTimeout(() => {
  console.log(`[dev]: ${devUrl}`);
}, 1000);

configList.map((config, index) => {
  config.output.sourcemap = true;

  if (index === 0) {
    config.plugins = [
      ...config.plugins,
      ...[
        serve({
          port: PORT,
          contentBase: [resolveFile("")],
        }),
      ],
    ];
  }

  return config;
});

module.exports = configList;
```

- `生产模式`配置基本 `./build/rollup.config.prod.js`

```js
const { uglify } = require("rollup-plugin-uglify");
const configList = require("./rollup.config");

const resolveFile = function (filePath) {
  return path.join(__dirname, "..", filePath);
};

configList.map((config, index) => {
  config.output.sourcemap = false;
  config.plugins = [...config.plugins, ...[uglify()]];

  return config;
});

module.exports = configList;
```

- 在`./package.json`配置编译执行脚本

```
{
  "scripts": {
    "build": "node_modules/.bin/rollup -c ./build/rollup.config.prod.js",
    "dev": "node_modules/.bin/rollup -w -c ./build/rollup.config.dev.js"
  },
}
```

### 步骤 3: 待编译 ES6 源码

- 源码路径 `./src/index.js`
- 源码内容

```js
import $ from "jquery";

const text = "this is append dom";
const dom = `<p>${text}</p>`;

$("body").append(dom);

console.log("render end!");
```

### 步骤 4: 编译结果

- 在项目目录下执行 `开发模式` `npm run dev`
- 编译结果在目录 `./dist/` 下

### 步骤 5: 浏览器查看结果

- example 目录`./example/index.html`
- example 源码

```html
<html>
  <head>
    <script src="https://cdn.bootcss.com/babel-polyfill/6.26.0/polyfill.js"></script>
  </head>
  <body>
    <p>hello world</p>
    <script src="./../dist/index.js"></script>
  </body>
</html>
```

- 访问 [http://127.0.0.1:3000/example/index.html](http://127.0.0.1:3000/example/index.html)
- 打开工作台 console 就会显示可运行结果

```sh
render end!
```
