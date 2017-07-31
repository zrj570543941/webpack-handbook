---
title: Lazy Loading
sort: 10
contributors:
  - iammerrick
  - chrisVillanueva
  - skipjack
related:
  - title: Lazy Loading ES2015 Modules in the Browser
    url: https://dzone.com/articles/lazy-loading-es2015-modules-in-the-browser
---

T> This guide is a small follow-up to [Code Splitting](/guides/code-splitting). If you have not yet read through that guide, please do so now.

Lazy, or "on demand", loading is a great way to optimize your site or application. This practice essentially involves splitting your code at logical breakpoints, and then loading it once the user has done something that requires, or will require, a new block of code. This speeds up the initial load of the application and lightens its overall weight as some blocks may never even be loaded.


## Example

Let's take the example from [Code Splitting](/guides/code-splitting#dynamic-imports) and tweak it a bit to demonstrate this concept even more. The code there does cause a separate chunk, `lodash.bundle.js`, to be generated and technically "lazy-loads" it as soon as the script is run. The trouble is that no user interaction is required to load the bundle -- meaning that every time the page is loaded, the request will fire. This doesn't help us too much and will impact performance negatively.

Let's try something different. We'll add an interaction to log some text to the console when the user clicks a button. However, we'll wait to load that code (`print.js`) until the interaction occurs for the first time. To do this we'll go back and rework the [final _Dynamic Imports_ example](/guides/code-splitting#dynamic-imports) from _Code Splitting_ and leave `lodash` in the main chunk.

__project__

``` diff
webpack-demo
|- package.json
|- webpack.config.js
|- /dist
|- /src
  |- index.js
+ |- print.js
|- /node_modules
```

__src/print.js__

``` js
console.log('The print.js module has loaded! See the network tab in dev tools...');

export default () => {
  console.log('Button Clicked: Here\'s "some text"!');
}
```

__src/index.js__

``` diff
+ import _ from 'lodash';
+
- async function getComponent() {
+ function component() {
    var element = document.createElement('div');
-   const _ = await import(/* webpackChunkName: "lodash" */ 'lodash');
+   var button = document.createElement('button');
+   var br = document.createElement('br');

+   button.innerHTML = 'Click me and look at the console!';
    element.innerHTML = _.join(['Hello', 'webpack'], ' ');
+   element.appendChild(br);
+   element.appendChild(button);
+
+   // Note that because a network request is involved, some indication
+   // of loading would need to be shown in a production-level site/app.
+   button.onclick = e => import(/* webpackChunkName: "print" */ './print').then(module => {
+     var print = module.default;
+
+     print();
+   });

    return element;
  }

- getComponent().then(component => {
-   document.body.appendChild(component);
- });
+ document.body.appendChild(component());
```

W> Note that when using `import()` on ES6 modules you must reference the `.default` property as it's the actual `module` object that will be returned when the promise is resolved.

Now let's run webpack and check out our new lazy-loading functionality:

``` bash
Hash: e0f95cc0bda81c2a1340
Version: webpack 3.0.0
Time: 1378ms
          Asset       Size  Chunks                    Chunk Names
print.bundle.js  417 bytes       0  [emitted]         print
index.bundle.js     548 kB       1  [emitted]  [big]  index
     index.html  189 bytes          [emitted]
   [0] ./src/index.js 742 bytes {1} [built]
   [2] (webpack)/buildin/global.js 509 bytes {1} [built]
   [3] (webpack)/buildin/module.js 517 bytes {1} [built]
   [4] ./src/print.js 165 bytes {0} [built]
    + 1 hidden module
Child html-webpack-plugin for "index.html":
       [2] (webpack)/buildin/global.js 509 bytes {0} [built]
       [3] (webpack)/buildin/module.js 517 bytes {0} [built]
        + 2 hidden modules
```


## Frameworks

Many frameworks and libraries have their own recommendations on how this should be accomplished within their methodologies. Here are a few examples:

- React: [Code Splitting and Lazy Loading](https://reacttraining.com/react-router/web/guides/code-splitting)
- Vue: [Lazy Load in Vue using Webpack's code splitting](https://alexjoverm.github.io/2017/07/16/Lazy-load-in-Vue-using-Webpack-s-code-splitting/)

***

> 原文：https://webpack.js.org/guides/lazy-loading/
