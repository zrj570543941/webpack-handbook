---
title: coffee-redux-loader
source: https://raw.githubusercontent.com/webpack-contrib/coffee-redux-loader/master/README.md
edit: https://github.com/webpack-contrib/coffee-redux-loader/edit/master/README.md
---
## Install

```bash
npm i -D coffee-redux-loader
```

## 用法

``` javascript
var exportsOfFile = require("coffee-redux-loader!./file.coffee");
// => return exports of executed and compiled file.coffee
```

如果你想要在 node 运行环境中使用，不要忘了 polyfill `require`。
请查看 `webpack` 文档。


## Maintainers

<table>
  <tbody>
    <tr>
      <td align="center">
        <img width="150" height="150"
        src="https://avatars3.githubusercontent.com/u/166921?v=3&s=150">
        </br>
        <a href="https://github.com/bebraw">Juho Vepsäläinen</a>
      </td>
      <td align="center">
        <img width="150" height="150"
        src="https://avatars2.githubusercontent.com/u/8420490?v=3&s=150">
        </br>
        <a href="https://github.com/d3viant0ne">Joshua Wiens</a>
      </td>
      <td align="center">
        <img width="150" height="150"
        src="https://avatars3.githubusercontent.com/u/533616?v=3&s=150">
        </br>
        <a href="https://github.com/SpaceK33z">Kees Kluskens</a>
      </td>
      <td align="center">
        <img width="150" height="150"
        src="https://avatars3.githubusercontent.com/u/3408176?v=3&s=150">
        </br>
        <a href="https://github.com/TheLarkInn">Sean Larkin</a>
      </td>
    </tr>
  <tbody>
</table>


[npm]: https://img.shields.io/npm/v/coffee-redux-loader.svg
[npm-url]: https://npmjs.com/package/coffee-redux-loader

[deps]: https://david-dm.org/webpack-contrib/coffee-redux-loader.svg
[deps-url]: https://david-dm.org/webpack-contrib/coffee-redux-loader

[chat]: https://img.shields.io/badge/gitter-webpack%2Fwebpack-brightgreen.svg
[chat-url]: https://gitter.im/webpack/webpack

***

> 原文：https://webpack.js.org/loaders/coffee-redux-loader/