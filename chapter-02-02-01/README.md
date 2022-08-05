# AMD 文件类型编译（上）

## 前言

一般项目中的 js 文件都有`IIFE`, `AMD`, `CommonJS`，`UMD`，四种模块化格式，具体的解释如下

- `IIFE` Imdiately Invoked Function Expression 立即执行函数
- `AMD` Asynchronous Module Definition 异步模块规范
- `CommonJS` CommonJS 规范，是 Node.js 的官方模块化规范
- `UMD`， Universal Module Definition 通用模块规范

本篇主要讲述 `AMD` 编译的配置，`AMD`规范就是将模块定义注册到缓存中，通过沙箱化和模块化去使用和执行。

定义模块

```js
defie("module-01", [], function () {
  return {
    init() {
      console.log("hello");
    },
  };
});
```

自执行模块

```js
define(["module-01"], function (mod1) {
  mod1.init();
});
```

但是有了 ES6+ 语法后，之前的模块化实现可以基于 ES6+语法，编写，最后的模块编译结果由`webpack`或者`rollup.js`等去处理，本篇就是讲述如何用`rollup.js`去将 ES6+语法编译成 `amd` 模式

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
    "build": "node_modules/.bin/rollup -c ./build/rollup.config.prod.js",
    "dev": "node_modules/.bin/rollup -w -c ./build/rollup.config.dev.js"
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
      format: "amd",
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

### 步骤 四: 编译结果

- 在项目目录下执行 `npm run build`
- 编译结果在目录 `./dist/` 下
- 编译成 ES5 结果为

```js
define(function () {
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
          asyncGeneratorStep(
            gen,
            resolve,
            reject,
            _next,
            _throw,
            "next",
            value
          );
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
        Object.defineProperties(
          target,
          Object.getOwnPropertyDescriptors(source)
        );
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
            _objectSpread2(
              _objectSpread2(_objectSpread2({}, obj1), obj2),
              obj3
            ),
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

  return index;
});
//# sourceMappingURL=index.js.map
```

### 步骤 5: 浏览器查看结果

- example 目录`./example/index.html`
- example 源码

```html
<html>
  <head>
    <meta charset="utf-8" />
    <script src="https://cdn.bootcss.com/babel-polyfill/6.26.0/polyfill.js"></script>
    <script
      data-main="main"
      src="https://cdn.bootcss.com/require.js/2.3.5/require.js"
    ></script>
  </head>
  <body>
    <p>hello world</p>
    <!-- <script src="./../dist/index.js"></script> -->
  </body>
</html>
```

- `require.js`引用的配置入口 `example/main.js`文件

```js
requirejs.config({
  baseUrl: "/",
  paths: {},
});

define(function (require) {
  var demo = require("dist/index");
  demo.init();
});
```

- 访问 [http://127.0.0.1:3000/example/index.html](http://127.0.0.1:3000/example/index.html)
- 打开工作台 console 就会显示可运行结果

```sh
[1, 2, 3, 4, 5, 6]
this is lib.js
{a: 1, b: 2, c: 3, d: 4}
>
```
