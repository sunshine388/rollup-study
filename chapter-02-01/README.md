# 多文件输入输出编译

## 前言

本篇主要讲述如何用 `rollup.js` 配置多个文件的编译输入和输出。

## 实现例子

- 编译输入多个 ES6+的 js 源文件 `src/index.js` 和 `src/lib/index.js`
- 编译输出多个 ES5 的 js 文件 `dist/index.js` 和 `dist/lib.js`

### 步骤一 安装

```sh
npm init -y

## 安装 rollup.js 基础模块
npm i --save-dev rollup

## 安装 rollup.js 编译本地开发服务插件
npm i --save-dev rollup-plugin-serve

## 安装 rollup.js 编译代码混淆插件
npm i --save-dev rollup-plugin-uglify

## 安装 rollup.js 编译ES6+的 babel 模块
npm i --save-dev @rollup/plugin-babel @babel/core @babel/preset-env
```

- 修改 package.json 文件 scripts

```
  "scripts": {
    "dev": "node_modules/.bin/rollup -w -c ./build/rollup.config.dev.js",
    "build": "node_modules/.bin/rollup -c ./build/rollup.config.prod.js"
  },
```

## 步骤二 rollup 配置

- 创建 `./build/rollup.config.js`

```js
const path = require("path");
const { babel } = require("@rollup/plugin-babel");

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
    plugins: [babel(babelOptions)],
  },
  {
    input: resolveFile("src/lib/index.js"),
    output: {
      file: resolveFile("dist/lib.js"),
      format: "cjs",
    },
    plugins: [babel(babelOptions)],
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
const PORT = 3001;

configList.map((config, index) => {
  config.output.sourcemap = true;

  if (index === 0) {
    config.plugins = [
      ...config.plugins,
      ...[
        serve({
          port: PORT,
          contentBase: [resolveFile("example"), resolveFile("dist")],
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
        const obj3 = { c: 3 };
        const obj4 = { d: 4 };
        const result = { ...obj1, ...obj2, ...obj3, ...obj4 };
        resolve(result);
      }, 1000);
    } catch (err) {
      reject(err);
    }
  });
}

export default demo;
```

- `./src/lib/index.js`源码内容

```js
console.log("this is lib.js");
```

### 步骤 四: 编译结果

- 在项目目录下执行`开发模式` `npm run dev`
- 在项目目录下执行`生产模式` `npm run build`
- 编译结果在目录 `./dist/` 下
- 编译结果分成
  - ES5 代码文件 `dist/index.js` 和 `dist/lib.js`
  - `生产模式` ES5 代码的生成会被`uglify`混淆压缩
  - `开发模式` 会生成源码的 sourceMap 文件 `./dist/index.js.map` 和 `./dist/lib.js.map`
- 插件服务启动了`3001` 端口

### 步骤 5: 浏览器查看结果

- example 目录`./example/index.html`

````
- 访问 [http://127.0.0.1:3001](http://127.0.0.1:3001)
- 打开工作台console 就会显示可运行结果

```sh
[1, 2, 3, 4, 5, 6]
this is lib.js
{a: 1, b: 2, c: 3, d: 4}
>
````
