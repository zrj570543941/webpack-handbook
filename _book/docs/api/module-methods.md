---
title: Module API - Methods
sort: 3
contributors:
  - skipjack
  - sokra
related:
  - title: CommonJS Wikipedia
    url: https://en.wikipedia.org/wiki/CommonJS
  - title: Asynchronous Module Definition
    url: https://en.wikipedia.org/wiki/Asynchronous_module_definition
---

This section covers all methods available in code compiled with webpack. When using webpack to bundle your application, you can pick from a variety of module syntax styles including [ES6](https://en.wikipedia.org/wiki/ECMAScript#6th_Edition_-_ECMAScript_2015), [CommonJS](https://en.wikipedia.org/wiki/CommonJS), and [AMD](https://en.wikipedia.org/wiki/Asynchronous_module_definition).

W> While webpack supports multiple module syntaxes, we recommend following a single syntax for consistency and to avoid odd behaviors/bugs. Here's [one example](https://github.com/webpack/webpack.js.org/issues/552) of mixing ES6 and CommonJS, however there are surely others.


## ES6 (Recommended)

Version 2 of webpack supports ES6 module syntax natively, meaning you can use `import` and `export` without a tool like babel to handle this for you. Keep in mind that you will still probably need babel for other ES6+ features. The following methods are supported by webpack:


### `import`

Statically `import` the `export`s of another module.

``` javascript
import MyModule from './my-module.js';
import { NamedExport } from './other-module.js';
```

W> The keyword here is __statically__. Normal `import` statement cannot be used dynamically within other logic or contain variables. See the [spec](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) for more information and `import()` below for dynamic usage.


### `export`

Export anything as a `default` or named export.

``` javascript
// Named exports
export var Count = 5;
export function Multiply(a, b) {
  return a * b;
}

// Default export
export default {
  // Some data...
}
```


### `import()`

`import('path/to/module') -> Promise`

Dynamically load modules. Calls to `import()` are treated as split points, meaning the requested module and it's children are split out into a separate chunk.

T> The [ES2015 Loader spec](https://whatwg.github.io/loader/) defines `import()` as method to load ES2015 modules dynamically on runtime.

``` javascript
if ( module.hot ) {
  import('lodash').then(_ => {
    // Do something with lodash (a.k.a '_')...
  })
}
```

W> This feature relies on [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) internally. If you use `import()` with older browsers, remember to shim `Promise` using a polyfill such as [es6-promise](https://github.com/stefanpenner/es6-promise) or [promise-polyfill](https://github.com/taylorhakes/promise-polyfill).

The spec for `import` doesn't allow control over the chunk's name or other properties as "chunks" are only a concept within webpack. Luckily webpack allows some special parameters via comments so as to not break the spec:

``` js
import(
  /* webpackChunkName: "my-chunk-name" */
  /* webpackMode: "lazy" */
  'module'
);
```

`webpackChunkName`: A name for the new chunk. Since webpack 2.6.0, the placeholders `[index]` and `[request]` are supported within the given string to an incremented number or the actual resolved filename respectively.

`webpackMode`: Since webpack 2.6.0, different modes for resolving dynamic imports can be specified. The following options are supported:

- `"lazy"` (default): Generates a lazy-loadable chunk for each `import()`ed module.
- `"lazy-once"`: Generates a single lazy-loadable chunk that can satisfy all calls to `import()`. The chunk will be fetched on the first call to `import()`, and subsequent calls to `import()` will use the same network response. Note that this only makes sense in the case of a partially dynamic statement, e.g. ``import(`./locales/${language}.json`)``, where there are multiple module paths that could potentially be requested.
- `"eager"`: Generates no extra chunk. All modules are included in the current chunk and no additional network requests are made. A `Promise` is still returned but is already resolved. In contrast to a static import, the module isn't executed until the call to `import()` is made.

T> Note that both options can be combined like so `/* webpackMode: "lazy-once", webpackChunkName: "all-i18n-data" */`. This is parsed as a JSON5 object without curly brackets.

W> Fully dynamic statements, such as `import(foo)`, __will fail__ because webpack requires at least some file location information. This is because `foo` could potentially be any path to any file in your system or project. The `import()` must contain at least some information about where the module is located, so bundling can be limited to a specific directory or set of files.

W> Every module that could potentially be requested on an `import()` call is included. For example, ``import(`./locale/${language}.json`)`` will cause every `.json` file in the `./locale` directory to be bundled into the new chunk. At run time, when the variable `language` has been computed, any file like `english.json` or `german.json` will be available for consumption.

W> The use of `System.import` in webpack [did not fit the proposed spec](https://github.com/webpack/webpack/issues/2163), so it was deprecated in webpack [2.1.0-beta.28](https://github.com/webpack/webpack/releases/tag/v2.1.0-beta.28) in favor of `import()`.


## CommonJS

The goal of CommonJS is to specify an ecosystem for JavaScript outside the browser. The following CommonJS methods are supported by webpack:


### `require`

``` javascript
require(dependency: String)
```

Synchronously retrieve the exports from another module. The compiler will ensure that the dependency is available in the output bundle.

``` javascript
var $ = require("jquery");
var myModule = require("my-module");
```

W> Using it asynchronously may not have the expected effect.


### `require.resolve`

``` javascript
require.resolve(dependency: String)
```

Synchronously retrieve a module's ID. The compiler will ensure that the dependency is available in the output bundle. See [`module.id`](/api/module-variables#module-id-commonjs-) for more information.

``` javascript
var id = require.resolve("dependency");
typeof id === "number";
id === 0 // if dependency is the entry point
id > 0 // elsewise
```

W> Module ID is a number in webpack (in contrast to NodeJS where it is a string -- the filename).


### `require.cache`

Multiple requires to the same module result in only one module execution and only one export. Therefore a cache in the runtime exists. Removing values from this cache cause new module execution and a new export.

W> This is only needed in rare cases for compatibility!

``` javascript
var d1 = require("dependency");
require("dependency") === d1
delete require.cache[require.resolve("dependency")];
require("dependency") !== d1
```

``` javascript
// in file.js
require.cache[module.id] === module
require("./file.js") === module.exports
delete require.cache[module.id];
require.cache[module.id] === undefined
require("./file.js") !== module.exports // in theory; in praxis this causes a stack overflow
require.cache[module.id] !== module
```


### `require.ensure`

W> `require.ensure()` is specific to webpack and superseded by `import()`.

``` javascript
require.ensure(dependencies: String[], callback: function(require), errorCallback: function(error), chunkName: String)
```

Split out the given `dependencies` to a separate bundle that that will be loaded asynchronously. When using CommonJS module syntax, this is the only way to dynamically load dependencies. Meaning, this code can be run within execution, only loading the `dependencies` if certain conditions are met.

W> This feature relies on [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) internally. If you use `require.ensure` with older browsers, remember to shim `Promise` using a polyfill such as [es6-promise](https://github.com/stefanpenner/es6-promise) or [promise-polyfill](https://github.com/taylorhakes/promise-polyfill).

``` javascript
var a = require('normal-dep');

if ( module.hot ) {
  require.ensure(['b'], function(require) {
    var c = require('c');

    // Do something special...
  });
}
```

The following parameters are supported in the order specified above:

- `dependencies`: An array of strings declaring all modules required for the code in the `callback` to execute.
- `callback`: A function that webpack will execute once the dependencies are loaded. An implementation of the `require` function is sent as a parameter to this function. The function body can use this to further `require()` modules it needs for execution.
- `errorCallback`: A function that is executed when webpack fails to load the dependencies.
- `chunkName`: A name given to the chunk created by this particular `require.ensure()`. By passing the same `chunkName` to various `require.ensure()` calls, we can combine their code into a single chunk, resulting in only one bundle that the browser must load.

W> Although the implementation of `require` is passed as an argument to the `callback` function, using an arbitrary name e.g. `require.ensure([], function(request) { request('someModule'); })` isn't handled by webpack's static parser. Use `require` instead, e.g. `require.ensure([], function(require) { require('someModule'); })`.



## AMD

Asynchronous Module Definition (AMD) is a JavaScript specification that defines an interface for writing and loading modules. The following AMD methods are supported by webpack:


### `define` (with factory)

``` javascript
define([name: String], [dependencies: String[]], factoryMethod: function(...))
```

If `dependencies` are provided, `factoryMethod` will be called with the exports of each dependency (in the same order). If `dependencies` are not provided, `factoryMethod` is called with `require`, `exports` and `module` (for compatibility!). If this function returns a value, this value is exported by the module. The compiler ensures that each dependency is available.

W> Note that webpack ignores the `name` argument.

``` javascript
define(['jquery', 'my-module'], function($, myModule) {
  // Do something with $ and myModule...

  // Export a function
  return function doSomething() {
    // ...
  };
});
```

W> This CANNOT be used in an asynchronous function.


### `define` (with value)

``` javascript
define(value: !Function)
```

This will simply export the provided `value`. The `value` here can be anything except a function.

``` javascript
define({
  answer: 42
});
```

W> This CANNOT be used in an async function.


### `require` (amd-version)

``` javascript
require(dependencies: String[], [callback: function(...)])
```

Similar to `require.ensure`, this will split the given `dependencies` into a separate bundle that will be loaded asynchronously. The `callback` will be called with the exports of each dependency in the `dependencies` array.

W> This feature relies on [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) internally. If you use AMD with older browsers (e.g. Internet Explorer 11), remember to shim `Promise` using a polyfill such as [es6-promise](https://github.com/stefanpenner/es6-promise) or [promise-polyfill](https://github.com/taylorhakes/promise-polyfill).

``` javascript
require(['b'], function(b) {
  var c = require("c");
});
```

W> There is no option to provide a chunk name.



## Labeled Modules

The internal `LabeledModulesPlugin` enables you to use the following methods for exporting and requiring within your modules:


### `export` label

Export the given `value`. The label can occur before a function declaration or a variable declaration. The function name or variable name is the identifier under which the value is exported.

``` javascript
export: var answer = 42;
export: function method(value) {
  // Do something...
};
```

W> Using it in an async function may not have the expected effect.


### `require` label

Make all exports from the dependency available in the current scope. The `require` label can occur before a string. The dependency must export values with the `export` label. CommonJS or AMD modules cannot be consumed.

__some-dependency.js__

``` javascript
export: var answer = 42;
export: function method(value) {
  // Do something...
};
```

``` javascript
require: 'some-dependency';
console.log(answer);
method(...);
```



## Webpack

Aside from the module syntaxes described above, webpack also allows a few custom, webpack-specific methods:


### `require.context`

``` javascript
require.context(directory:String, includeSubdirs:Boolean /* optional, default true */, filter:RegExp /* optional */)
```

Specify a whole group of dependencies using a path to the `directory`, an option to `includeSubdirs`, and a `filter` for more fine grained control of the mdoules included. These can then be easily `resolve`d later on.

```javascript
var context = require.context('components', true, /\.html$/);
var componentA = context.resolve('componentA');
```


### `require.include`

``` javascript
require.include(dependency: String)
```

Include a `dependency` without executing it. This can be used for optimizing the position of a module in the output chunks.

``` javascript
require.include('a');
require.ensure(['a', 'b'], function(require) { /* ... */ });
require.ensure(['a', 'c'], function(require) { /* ... */ });
```

This will result in following output:

- entry chunk: `file.js` and `a`
- anonymous chunk: `b`
- anonymous chunk: `c`

Without `require.include('a')` it would be duplicated in both anonymous chunks.


### `require.resolveWeak`

Similar to `require.resolve`, but this won't pull the `module` into the bundle. It's what is considered a "weak" dependency.

``` javascript
if(__webpack_modules__[require.resolveWeak('module')]) {
  // Do something when module is available...
}
if(require.cache[require.resolveWeak('module')]) {
  // Do something when module was loaded before...
}
```

***

> 原文：https://webpack.js.org/api/module-methods/
