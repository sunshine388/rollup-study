# 1.4 生产模式

## 前言

`生产模式` 就是项目正式上线的模式，前端代码`生产模式`主要有以下几点要素：

- 保证代码混淆，编译结果不可读
- 体积压缩
- 信息脱敏

因此，rollup.js 的在`生产模式`下编译后的代码要有以下几点要求：

- 代码 uglify
- 关闭 sourceMap
- `npm run build` 启动执行 `生产模式`
- `npm run dev` 启动执行 `开发模式`

### 步骤一 安装

```sh
  npm init -y

  ## 安装 rollup.js 基础模块
  npm i --save-dev rollup
  ## 安装 rollup.js 编译ES6+的 babel 模块
  npm i --save-dev @rollup/plugin-babel @babel/core @babel/preset-env
  ## 安装 rollup.js 编译本地开发服务插件
  npm i --save-dev rollup-plugin-serve
  ## 安装 rollup.js 编译代码混淆插件
  npm i --save-dev rollup-plugin-uglify
  ## 安装 rollup-plugin-delete 删除指定目录
  npm i --save-dev rollup-plugin-delete
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
const del = require("rollup-plugin-delete");

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
    del({ targets: "dist/*" }),
    babel({
      presets: ["@babel/preset-env"],
    }),
  ],
};
```

- `开发模式`配置基本 `./build/rollup.config.dev.js`

```js
const path = require("path");
const serve = require("rollup-plugin-serve");
const config = require("./rollup.config");

const resolveFile = function (filePath) {
  return path.join(__dirname, "..", filePath);
};
const PORT = 3001;

config.output.sourcemap = true;
config.plugins = [
  ...config.plugins,
  ...[
    serve({
      port: PORT,
      // contentBase: [resolveFile('')]
      contentBase: [resolveFile("example"), resolveFile("dist")],
    }),
  ],
];

module.exports = config;
```

- `生产模式`配置基本 `./build/rollup.config.prod.js`

```js
const { uglify } = require("rollup-plugin-uglify");
const config = require("./rollup.config");

config.output.sourcemap = false;
config.plugins = [...config.plugins, ...[uglify()]];

module.exports = config;
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

- 在项目目录下执行`开发模式` `npm run dev`
- 在项目目录下执行`生产模式` `npm run build`
- 编译结果在目录 `./dist/` 下
- 编译结果分成
  - ES5 代码文件 `./dist/index.js`
  - `生产模式` ES5 代码的生成会被`uglify`混淆压缩
  - `开发模式` 会生成源码的 sourceMap 文件 `./dist/index.js.map`
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
