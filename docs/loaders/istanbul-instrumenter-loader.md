---
title: istanbul-instrumenter-loader
source: https://raw.githubusercontent.com/webpack-contrib/istanbul-instrumenter-loader/master/README.md
edit: https://github.com/webpack-contrib/istanbul-instrumenter-loader/edit/master/README.md
---
## 安装

```bash
npm i -D istanbul-instrumenter-loader
```

## <a href="https://webpack.js.org/concepts/loaders">用法</a>

### `参考`

* [karma-webpack](https://github.com/webpack/karma-webpack)
* [karma-coverage-istanbul-reporter](https://github.com/mattlewis92/karma-coverage-istanbul-reporter)

### `结构`

```
├─ src
│ |– components
│ | |– bar
│ | │ |─ index.js
│ | |– foo/
│     |– index.js
|– test
| |– src
| | |– components
| | | |– foo
| | | | |– index.js
```

为生成所有组件（包括你没写测试的那些）的代码覆盖率报告，你需要 require 所有**业务**和**测试**的代码。相关内容在 [karma-webpack 其他用法](https://github.com/webpack/karma-webpack#alternative-usage)中有涉及

**test/index.js**
```js
// requires 所有在 `project/test/src/components/**/index.js` 中的测试
const tests = require.context('./src/components/', true, /index\.js$/);

tests.keys().forEach(tests);

// requires 所有在 `project/src/components/**/index.js` 中的组件
const components = require.context('../src/components/', true, /index\.js$/);

components.keys().forEach(components);
```

> ℹ️  以下为 `karma` 的唯一`入口`起点文件

**karma.conf.js**
```js
config.set({
  ...
  files: [
    'test/index.js'
  ],
  preprocessors: {
    'test/index.js': 'webpack'
  },
  webpack: {
    ...
    module: {
      rules: [
        // 用 Istanbul 只监测业务代码
        {
          test: /\.js$/,
          use: { loader: 'istanbul-instrumenter-loader' },
          include: path.resolve('src/components/')
        }
      ]
    }
    ...
  },
  reporters: [ 'progress', 'coverage-istanbul' ],
  coverageIstanbulReporter: {
    reports: [ 'text-summary' ],
    fixWebpackSourcePaths: true
  }
  ...
});
```

### 使用 `Babel`

You must run the instrumentation as a post step

```js
{
  test: /\.js$|\.jsx$/,
  use: {
    loader: 'istanbul-instrumenter-loader',
    options: { esModules: true }
  },
  enforce: 'post',
  exclude: /node_modules|\.spec\.js$/,
}
```

## <a href="https://github.com/istanbuljs/istanbuljs/blob/master/packages/istanbul-lib-instrument/api.md#instrumenter">Options</a>

此 loader 支持 `istanbul-lib-instrument` 的所有配置选项

|Name|Type|Default|Description|
|:--:|:--:|:-----:|:----------|
|**`debug`**|`{Boolean}`|`false`|Turn on debugging mode|
|**`compact`**|`{Boolean}`|`true`|Generate compact code|
|**`autoWrap`**|`{Boolean}`|`false`|Set to `true` to allow return statements outside of functions|
|**`esModules`**|`{Boolean}`|`false`|Set to `true` to instrument ES2015 Modules|
|**`coverageVariable`**|`{String}`|`__coverage__`|Name of global coverage variable|
|**`preserveComments`**|`{Boolean}`|`false`|Preserve comments in `output`|
|**`produceSourceMap`**|`{Boolean}`|`false`|Set to `true` to produce a source map for the instrumented code|
|**`sourceMapUrlCallback`**|`{Function}`|`null`|A callback function that is called when a source map URL is found in the original code. This function is called with the source filename and the source map URL|

```js
{
  test: /\.js$/,
  use: {
    loader: 'istanbul-instrumenter-loader',
    options: {...options}
  }
}
```

## Maintainers

<table>
  <tbody>
    <tr>
      <td align="center">
        <img width="150" height="150"
        src="https://avatars.githubusercontent.com/u/266822?v=3&s=150">
        </br>
        <a href="https://github.com/deepsweet">Kir Belevich</a>
      </td>
      <td align="center">
        <a href="https://github.com/bebraw">
          <img width="150" height="150" src="https://github.com/bebraw.png?v=3&s=150">
          </br>
          Juho Vepsäläinen
        </a>
      </td>
      <td align="center">
        <a href="https://github.com/d3viant0ne">
          <img width="150" height="150" src="https://github.com/d3viant0ne.png?v=3&s=150">
          </br>
          Joshua Wiens
        </a>
      </td>
      <td align="center">
        <a href="https://github.com/michael-ciniawsky">
          <img width="150" height="150" src="https://github.com/michael-ciniawsky.png?v=3&s=150">
          </br>
          Michael Ciniawsky
        </a>
      </td>
      <td align="center">
        <a href="https://github.com/mattlewis92">
          <img width="150" height="150" src="https://github.com/mattlewis92.png?v=3&s=150">
          </br>
          Matt Lewis
        </a>
      </td>
    </tr>
  <tbody>
</table>


[npm]: https://img.shields.io/npm/v/istanbul-instrumenter-loader.svg
[npm-url]: https://npmjs.com/package/istanbul-instrumenter-loader

[node]: https://img.shields.io/node/v/istanbul-instrumenter-loader.svg
[node-url]: https://nodejs.org

[deps]: https://david-dm.org/webpack-contrib/istanbul-instrumenter-loader.svg
[deps-url]: https://david-dm.org/webpack-contrib/istanbul-instrumenter-loader

[tests]: http://img.shields.io/travis/webpack-contrib/istanbul-instrumenter-loader.svg
[tests-url]: https://travis-ci.org/webpack-contrib/istanbul-instrumenter-loader

[cover]: https://codecov.io/gh/webpack-contrib/istanbul-instrumenter-loader/branch/master/graph/badge.svg
[cover-url]: https://codecov.io/gh/webpack-contrib/istanbul-instrumenter-loader

[chat]: https://badges.gitter.im/webpack/webpack.svg
[chat-url]: https://gitter.im/webpack/webpack

***

> 原文：https://webpack.js.org/loaders/istanbul-instrumenter-loader/
