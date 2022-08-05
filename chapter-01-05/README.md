# 1.5 Node.js 使用模式

## 前言

本篇主要讲述用 rollup.js 的 API 在 Node.js 代码中执行编译代码。

## 实现例子

- 利用 rollup.js 的 API
- 在 Node.js 脚本中编译

### 步骤一 安装

```sh
npm init -y

## 安装 rollup.js 基础模块
npm i --save-dev rollup
## 安装 rollup.js 编译ES6+的 babel 模块
npm i --save-dev @rollup/plugin-babel @babel/core @babel/preset-env
```

- 修改 package.json 文件 scripts

```
  "scripts": {
    "build": "node ./build/build.js"
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

module.exports = {
  input: resolveFile("src/index.js"),
  output: {
    file: resolveFile("dist/index.js"),
    format: "umd",
  },
  plugins: [
    babel({
      presets: ["@babel/preset-env"],
    }),
  ],
};
```

- Node.js 调用 rollup 编译执行 `./build/build.js`

```js
const rollup = require("rollup");
const config = require("./rollup.config");

const inputOptions = config;
const outputOptions = config.output;

async function build() {
  // create a bundle
  const bundle = await rollup.rollup(inputOptions);
  console.log(`[INFO] 开始编译 ${inputOptions.input}`);
  // generate code and a sourcemap
  const { code, map } = await bundle.generate(outputOptions);
  console.log(`[SUCCESS] 编译结束 ${outputOptions.file}`);
  // or write the bundle to disk
  await bundle.write(outputOptions);
}

build();
```

## 步骤三 创建待编译 ES6 源码

- 源码路径 `./src/index.js`

- 源码路径 `./src/lib/demo.js`

### 步骤 四: 编译结果

- 在项目目录下执行 `npm run build`
- 编译结果在目录 `./dist/` 下
