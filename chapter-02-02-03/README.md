# CommonJS 模块类型编译

## 前言

CommonJS 规范，是 JavaScript 模块化的规范，目的是为了解决 JavaScript 的作用域和模块化的问题。主要内容是，模块必须通过 `module.exports`或`exports` 导出模块对外的变量或接口，通过`require` 来导入其他模块的输出到当前模块作用域中。因为目前 Node.js 还没完全支持和兼容`ES6+`的`import/exports`模块化规范，所以 CommonJS 目前是 Node.js 最安全通用的模块化方案。

这就出现一个需求，如果用`ES6+`的`import/exports`模块化规范开发`Node.js`代码，就需要编译成 Node.js 目前完全支持的 CommonJS 模块。这一篇就主要讲述`rollup.js`的 CommonJS 模块类型编译

## 实现例子

- 编译

demo 例子

[https://github.com/chenshenhai/rollupjs-note/blob/master/demo/chapter-02-02-03/](https://github.com/chenshenhai/rollupjs-note/blob/master/demo/chapter-02-02-03/)

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
├── build # rollup.js 编译脚本目录
│   ├── rollup.config.dev.js
│   ├── rollup.config.js
│   └── rollup.config.prod.js
├── dist # 编译结果目录
│   ├── index.js
│   └── index.js.map
├── example # 例子
│   └── index.js
├── package.json
└── src # 待编译源代码目录
    ├── index.js
    └── lib
        └── demo.js
```

安装对应编译的 npm 模块

```sh
## 安装 rollup.js 基础模块
npm i --save-dev rollup

## 安装 rollup.js 编译本地开发服务插件
npm i --save-dev rollup-plugin-serve

## 安装 rollup.js 编译代码混淆插件
npm i --save-dev rollup-plugin-uglify

## 安装 rollup.js 编译ES6+的 babel 模块
npm i --save-dev @rollup/plugin-babel @babel/core @babel/preset-env
```

- `rollup` 模块是 rollup 编译的核心模块

### 步骤 2: rollup 配置

- 编译基本配置 `./build/rollup.config.js`

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
      format: "cjs",
    },

    plugins: [babel(babelOptions)],
  },
];
```

- `开发模式`配置基本 `./build/rollup.config.dev.js`

```js
const configList = require("./rollup.config");

configList.map((config, index) => {
  config.output.sourcemap = true;

  return config;
});

module.exports = configList;
```

- `生产模式`配置基本 `./build/rollup.config.prod.js`

```js
const { uglify } = require("rollup-plugin-uglify");
const configList = require("./rollup.config");

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
    "dev": "node_modules/.bin/rollup -w -c ./build/rollup.config.dev.js",
    "example": "node ./example/index",
  },
}
```

### 步骤 3: 待编译 ES6 源码

- 源码 `./src/index.js`

```js
import demo from "./lib/demo";

export default {
  init() {
    const arr1 = [1, 2, 3];
    const arr2 = [4, 5, 6];
    console.log([...arr1, ...arr2]);

    async function initDemo() {
      let data = await demo();
      console.log(data);
    }

    initDemo();
  },
};
```

- 源码 `./src/lib/demo.js`

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

### 步骤 4: 编译结果

- 在项目目录下执行 开发模式 `npm run dev`
- 编译结果在目录 `./dist/` 下
- 编译成 ES5 结果为

```js
"use strict";

function asyncGeneratorStep(gen, resolve, reject, _next, _throw, key, arg) {
  try {
    var info = gen[key](arg);
    var value = info.value;
  } catch (error) {
    reject(error);
    return;
  }

  if (info.done) {
    resolve(value);
  } else {
    Promise.resolve(value).then(_next, _throw);
  }
}

function _asyncToGenerator(fn) {
  return function () {
    var self = this,
      args = arguments;
    return new Promise(function (resolve, reject) {
      var gen = fn.apply(self, args);

      function _next(value) {
        asyncGeneratorStep(gen, resolve, reject, _next, _throw, "next", value);
      }

      function _throw(err) {
        asyncGeneratorStep(gen, resolve, reject, _next, _throw, "throw", err);
      }

      _next(undefined);
    });
  };
}

function _defineProperty(obj, key, value) {
  if (key in obj) {
    Object.defineProperty(obj, key, {
      value: value,
      enumerable: true,
      configurable: true,
      writable: true,
    });
  } else {
    obj[key] = value;
  }

  return obj;
}

function ownKeys(object, enumerableOnly) {
  var keys = Object.keys(object);

  if (Object.getOwnPropertySymbols) {
    var symbols = Object.getOwnPropertySymbols(object);
    if (enumerableOnly)
      symbols = symbols.filter(function (sym) {
        return Object.getOwnPropertyDescriptor(object, sym).enumerable;
      });
    keys.push.apply(keys, symbols);
  }

  return keys;
}

function _objectSpread2(target) {
  for (var i = 1; i < arguments.length; i++) {
    var source = arguments[i] != null ? arguments[i] : {};

    if (i % 2) {
      ownKeys(Object(source), true).forEach(function (key) {
        _defineProperty(target, key, source[key]);
      });
    } else if (Object.getOwnPropertyDescriptors) {
      Object.defineProperties(target, Object.getOwnPropertyDescriptors(source));
    } else {
      ownKeys(Object(source)).forEach(function (key) {
        Object.defineProperty(
          target,
          key,
          Object.getOwnPropertyDescriptor(source, key)
        );
      });
    }
  }

  return target;
}

function demo() {
  return new Promise(function (resolve, reject) {
    try {
      setTimeout(function () {
        var obj1 = {
          a: 1,
        };
        var obj2 = {
          b: 2,
        };
        var obj3 = {
          c: 3,
        };
        var obj4 = {
          d: 4,
        };

        var result = _objectSpread2(
          _objectSpread2(_objectSpread2(_objectSpread2({}, obj1), obj2), obj3),
          obj4
        );

        resolve(result);
      }, 1000);
    } catch (err) {
      reject(err);
    }
  });
}

var index = {
  init: function init() {
    var arr1 = [1, 2, 3];
    var arr2 = [4, 5, 6];
    console.log([].concat(arr1, arr2));

    function initDemo() {
      return _initDemo.apply(this, arguments);
    }

    function _initDemo() {
      _initDemo = _asyncToGenerator(
        /*#__PURE__*/ regeneratorRuntime.mark(function _callee() {
          var data;
          return regeneratorRuntime.wrap(function _callee$(_context) {
            while (1) {
              switch ((_context.prev = _context.next)) {
                case 0:
                  _context.next = 2;
                  return demo();

                case 2:
                  data = _context.sent;
                  console.log(data);

                case 4:
                case "end":
                  return _context.stop();
              }
            }
          }, _callee);
        })
      );
      return _initDemo.apply(this, arguments);
    }

    initDemo();
  },
};

module.exports = index;
//# sourceMappingURL=index.js.map
```

### 步骤 5: 测试例子

- example 目录`./example/index.js`
- example 源码

```js
require("@babel/polyfill");
const demo = require("./../dist/index");

demo.init();
```

- 测试例子 `npm run example`

```sh
> node ./example/index

[ 1, 2, 3, 4, 5, 6 ]
{ a: 1, b: 2, c: 3, d: 4 }
```

## 后记

本篇主要讲述 `rollup.js` 怎么配置编译 CommonJS 模块类型，源码中使用了 `async/await` 的语法，经过 `babel` 编译成 ES5 代码是需要加入`@babel/polyfill` 模块的。由于本篇只讲述怎么编译 CommomJS 模块类型，所以把`@babel/polyfill`npm 模块引用放到`example/index.js`里进行测试。后续有一篇是讲述 `rollup.js`编译 npm 模块的引用。
