# Typescript 编译

## 前言

本篇主要介绍 `rollup.js` 官方提供的编译 `Typescript` 的插件。

## 实现例子

- 编译 Typescript

demo 例子

[https://github.com/chenshenhai/rollupjs-note/blob/master/demo/chapter-06-01/](https://github.com/chenshenhai/rollupjs-note/blob/master/demo/chapter-06-01/)

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
├── build ## 编译脚本
│   ├── rollup.config.dev.js
│   ├── rollup.config.js
│   └── rollup.config.prod.js
├── dist ## 编译结果
│   ├── index.js
│   └── index.js.map
├── example ## 例子
│   └── index.html
├── package.json
└── src ## 待编译源码
    └── index.ts
```

安装对应编译的 npm 模块

```sh
## 安装 rollup.js 基础模块
npm i --save-dev rollup @rollup/plugin-buble

## 安装 rollup.js 编译本地开发服务插件
npm i --save-dev rollup-plugin-serve

## 安装 rollup.js 编译代码混淆插件
npm i --save-dev rollup-plugin-uglify

## 安装 rollup.js 编译 Typescript 代码的插件模块
npm i --save-dev @rollup/plugin-typescript typescript

```

### 步骤 2: rollup 配置

- 编译基本配置 `./build/rollup.config.js`

```js
const path = require("path");
const buble = require("@rollup/plugin-buble");
const typescript = require("@rollup/plugin-typescript");

const resolveFile = function (filePath) {
  return path.join(__dirname, "..", filePath);
};

module.exports = [
  {
    input: resolveFile("src/index.ts"),
    output: {
      file: resolveFile("dist/index.js"),
      format: "iife",
      name: "helloworld",
    },
    plugins: [typescript(), buble()],
  },
];
```

- `开发模式`配置基本 `./build/rollup.config.dev.js`

```js
process.env.NODE_ENV = "development";

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
process.env.NODE_ENV = "production";

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

- 源码内容 `./src/index.ts`

```js
export default async function (n: number) {
  while (--n) {
    await delay(10, n);
  }
}

function delay(interval: number, num: number) {
  return new Promise((resolve) =>
    setTimeout(() => {
      console.log(num);
      resolve();
    }, interval)
  );
}
```

### 步骤 4: 编译结果

- 在项目目录下执行 `开发模式` `npm run dev`
- 编译结果在目录 `./dist/` 下

### 步骤 5: 浏览器查看结果

- example 源码`./example/index.html`

```html
<html>
  <head></head>
  <body>
    <p>hello world</p>
    <script src="./../dist/index.js"></script>
    <script>
      helloworld(8);
    </script>
  </body>
</html>
```

- 访问 [http://127.0.0.1:3000/example/index.html](http://127.0.0.1:3000/example/index.html)
- 打开工作台 console 就会显示可运行结果

```sh
7
6
5
4
3
2
1
>
```
