---
title: Babel详解
date: 2019-08-15 20:49:42
tags: [babel]
id: l5
categories: 学习笔记
---

## Babel 原理之三部曲

> 会编译原理的人真的可以为所欲为

### Parse（解析）

> 通过@babel/parser（原 babylon），将 js 代码根据[ESTree](https://github.com/estree/estree)的规范解析成 **AST**（`Abstract Syntax Tree`）

如下将

```js
const sum = () => {}
```

解析为 AST 树

```js
var parse = require("@babel/parser").parse
var code = `const sum=()=>{}`
var ast = parse(code)
```

AST:

```js
{
    "type": "File",
    "program": {//根节点，完整的抽象语法树
        "type": "Program",
        "sourceType": "script",
        "interpreter": null,
        "body": [
            {
                "type": "VariableDeclaration",//变量声明
                "declarations": [
                    {
                        "type": "VariableDeclarator",//变量声明
                        "id": {
                            "type": "Identifier",//标识符 变量名称等
                            "name": "sum"
                        },
                        "init": {//初始的表达式
                            "type": "ArrowFunctionExpression",//箭头函数
                            "id": null,
                            "generator": false,
                            "async": false,
                            "params": [],
                            "body": {
                                "type": "BlockStatement",//块语句节点
                                "body": [],
                                "directives": []
                            }
                        }
                    }
                ],
                "kind": "const"//类型
            }
        ],
        "directives": []
    },
    "comments": []
}
```

### Transform（转换）

> babel 并不具备转换的功能，转换都是通过转换插件来完成的  
> 官方插件以@babel/plugin-transform（正式）@babel/plugin-proposal（提案）开头

#### plugins

> 插件

-   @babel/plugin-proposal-function-bind [Stage 0]
-   @babel/plugin-proposal-decorators [Stage 2]
-   @babel/plugin-proposal-class-properties[Stage 3]
-   ...

#### presets

> 常用的插件的集合

常用的

-   @babel/preset-env 正式批准的内容，现包含（es2015、es2016、es2017...）
-   @babel/preset-react
-   @babel/preset-typescript
-   ...

#### 执行顺序

-   先 plugins 再 presets
-   plugins 正序
-   presets 倒序

### Generator（生成）

> 通过@babel/generator，将 AST 生成代码

AST 生成代码

```js
var generate = require("@babel/generator").default
var code = generate(ast).code
```

## Babel 配套工具

### @babel/cli

> 命令行工具

### @babel/core

> babel 核心，其中包含之前说的 @babel/parser 和@babel/generator

### @babel/register

> 改写 require 命令，对 require 加载对文件进行转码

在文件里面加载 require('babel-register')，不作用于当前的文件

### @babel/polyfill

> 意为垫片或补丁包

babel 插件只对新语法做转换，而不转换新对 API 例如：Generator、Set、Maps、Proxy、Reflect、Symbol、Promise 等全局对象，或者是定义在全局对象上的方法，Array.from、Object.assign 等。

`@babel/polyfill` 其实就是`core-js`、`regenerator-runtime`

babel7.4 之前直接使用`@babel/polyfill`

polyfill 的方式根据 useBuiltIns 的不同可以分为三种，即 `entry`, `usage` ,`false`，可配合 preset-env 使用

待 babel 的源码如下

```js
const sum = () => {}
const pro = new Promise(() => {})
```

-   entry：引入补丁兼容浏览器配置的补丁

```js
"use strict";
require("core-js/modules/es6.array.reduce");

require("core-js/modules/es6.array.reduce-right");
...
require("core-js/modules/es6.promise");
...

var sum = function sum() {};

var pro = new Promise(function () {});
```

-   usage：引入代码中用到的补丁

```js
"use strict"
require("core-js/modules/es6.promise")
var sum = function sum() {}
var pro = new Promise(function() {})
```

-   false：引入所有补丁

这样就解决了新 API 的问题，但是这种方式又带了另一种问题，就是全局污染，那就引出了另一个解决这个问题的插件` @babel/runtime``@babel/plugin-transform-runtime `

### @babel/runtime、@babel/plugin-transform-runtime

-   `@babel/runtime`实际导入项目代码的功能模块

-   `@babel/plugin-transform-runtime`构建过程的代码转换

缺点，不能对实例进行转换

```js
"use strict"

var _interopRequireDefault = require("@babel/runtime-corejs2/helpers/interopRequireDefault")

require("core-js/modules/es7.array.includes")

require("core-js/modules/es6.string.includes")

var _promise = _interopRequireDefault(
    require("@babel/runtime-corejs2/core-js/promise")
)

var sum = function sum() {}

var pro = new _promise.default(function() {})
"hello".includes("h") //还是通过polyfill 用全局变量的方式实现
```

不能对实例上的方法打沙盒补丁该怎么办呢，这时就出现了 babel7.4+
废弃 polyfill，单独引入`core-js`,`regenerator-runtime`，而且 core-js3 也解决了实例的方法的问题

```js
"use strict"

var _interopRequireDefault = require("@babel/runtime-corejs3/helpers/interopRequireDefault")

var _includes = _interopRequireDefault(
    require("@babel/runtime-corejs3/core-js-stable/instance/includes")
)

var _promise = _interopRequireDefault(
    require("@babel/runtime-corejs3/core-js-stable/promise")
)

var _context

var pro = new _promise["default"](function() {})
;(0, _includes["default"])((_context = "hello")).call(_context, "h")
```