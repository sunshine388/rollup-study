### 步骤一 安装

```sh
  npm init -y
  npm install rollup @rollup/plugin-babel @babel/core @babel/preset-env --D
```

- 修改 package.json 文件 scripts

```
  "scripts": {
  "build": "node_modules/.bin/rollup -c ./build/rollup.config.js"
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

const resolve = function (filePath) {
  return path.join(__dirname, "..", filePath);
};

module.exports = {
  input: resolve("src/index.js"),
  output: {
    file: resolve("dist/index.js"),
    format: "umd",
  },
  plugins: [
    babel({
      presets: ["@babel/preset-env"],
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

- 在项目目录下执行 `npm run build`
- 编译结果在目录 `./dist/` 下
- 编译成 ES5 结果为
- 注意：页面使用的时候要引入 `babel-polyfill.js`或者打包的时候把`babel-polyfill` 模块引用进入项目中

```js
(function (factory) {
  typeof define === "function" && define.amd ? define(factory) : factory();
})(function () {
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

          var result = _objectSpread2(_objectSpread2({}, obj1), obj2);

          resolve(result);
        }, 1000);
      } catch (err) {
        reject(err);
      }
    });
  }

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
});
```

### 步骤 五: 浏览器查看结果

- example 目录`./example/index.html`
- example 源码

```html
<html>
  <head>
    <script src="https://cdn.bootcss.com/babel-polyfill/6.26.0/polyfill.js"></script>
  </head>
  <body>
    <script src="./../dist/index.js"></script>
  </body>
</html>
```

- 直接用 chrome 打开项目本地的 `./example/index.html`
- 打开工作台 console 就会显示可运行结果

```sh
[1, 2, 3, 4, 5, 6]
{a: 1, b: 2}
>
```
