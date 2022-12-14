# IIFE 模块类型编译

## 前言

IIFE，是立即调用函数表达式（Immediately-invoked function expression）的缩写。简单来说就是，在定义时就会立即执行的 JavaScript 函数。

`IIFE`规范编译结果有以下特点

- 模块以执行函数封装
- 将模块挂载到全局对象 `window`上

例如

```js
(function () {
  var demo = "iife";
  window.demo = demo;
})();
```

## 实现例子

demo 例子

[https://github.com/chenshenhai/rollupjs-note/blob/master/demo/chapter-02-02-04/](https://github.com/chenshenhai/rollupjs-note/blob/master/demo/chapter-02-02-04/)

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
      format: "iife",
      name: "Demo",
      amd: {
        id: "lib/demo",
      },
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
const devIIFEPath = path.join("example", "/index.html");
const devIIFEUrl = `${devSite}/${devIIFEPath}`;

setTimeout(() => {
  console.log(`[dev IIFE]: ${devIIFEUrl}`);
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

- 在项目目录下执行 `生产模式` `npm run dev`
- 编译结果在目录 `./dist/` 下
- 编译成 ES5 结果为

```js
var Demo = (function () {
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
})();
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
  </head>
  <body>
    <p>hello world</p>
    <script src="./../dist/index.js"></script>
    <script>
      window.Demo.init();
    </script>
  </body>
</html>
```

- 访问 [http://127.0.0.1:3000/example/index.html](http://127.0.0.1:3000/example/index.html)
- 打开工作台 console 就会显示可运行结果

```sh
[1, 2, 3, 4, 5, 6]
{a: 1, b: 2, c: 3, d: 4}
```
