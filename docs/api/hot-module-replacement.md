---
title: 模块热替换(Hot Module Replacement)
contributors:
  - sokra
  - skipjack
related:
  - title: 概念 - 模块热替换(Hot Module Replacement)
    url: /concepts/hot-module-replacement
  - title: 指南 - 模块热替换(Hot Module Replacement)
    url: /guides/hot-module-replacement
---

如果已经通过 [`HotModuleReplacementPlugin`](/plugins/hot-module-replacement-plugin) 启用了[模块热替换(Hot Module Replacement)](/concepts/hot-module-replacement)，则它的接口将被暴露在 [`module.hot` 属性](/api/module-variables#module-hot-webpack-specific-)下面。通常，用户将检查界面是否可访问，然后再开始使用它。举个例子，你可以这样 `accept` 一个更新的模块：

``` js
if (module.hot) {
  module.hot.accept('./library.js', function() {
    // 使用更新过的 library 模块执行某些操作...
  })
}
```

支持以下方法……


### `accept`

接受(accept)给定`依赖模块`的更新，并触发一个 `回调函数` 来对这些更新做出响应。

``` js
module.hot.accept(
  dependencies, // 可以是一个字符串或字符串数组
  callback // 用于在模块更新后触发的函数
)
```


### `decline`

拒绝给定`依赖模块`的更新，使用 `'decline'` 方法强制更新失败。

``` js
module.hot.decline(
  dependencies // 可以是一个字符串或字符串数组
)
```


### `dispose`（或 `addDisposeHandler`）

添加一个处理函数，在当前模块代码被替换时执行。此函数应该用于销毁你声明或创建的任何持久资源。如果要将状态传入到更新过的模块，请添加给定 `data` 参数。更新后，此对象在更新之后可通过 `module.hot.data` 调用。

``` js
module.hot.dispose(data => {
  // 清理并将 data 传递到更新后的模块……
})
```


### `removeDisposeHandler`

删除由 `dispose` 或 `addDisposeHandler` 添加的回调函数。

``` js
module.hot.removeDisposeHandler(callback)
```


### `status`

取得模块热替换进程的当前状态。

``` js
module.hot.status() // 返回以下字符串之一……
```

| Status      | Description                                                                            |
| ----------- | -------------------------------------------------------------------------------------- |
| idle        | 该进程正在等待调用 `check`（见下文） |
| check       | 该进程正在检查以更新 |
| watch       | 该进程处于观察模式，并将自动通知有关更改 |
| watch-delay | 在初始更改后，延迟指定时间，再应用任何更新 |
| prepare     | 该进程正在准备更新（例如，下载好更新的模块） |
| ready       | 此更新已准备并可用 |
| dispose     | 该进程正在调用将被替换模块的 `dispose` 处理函数 |
| apply       | 该进程正在调用 `accept` 处理函数，并重新执行自我接受(self-accepted)的模块 |
| abort       | 更新已中止，但系统仍处于以前的状态 |
| fail        | 更新已抛出异常，系统状态已经损坏 |


### `check`

测试所有加载的模块以进行更新，如果有更新，则应用它们。

``` js
module.hot.check(autoApply, (error, outdatedModules) => {
  // 捕获错误和超时的模块……
})
```

`autoApply` 参数可以是布尔值，也可以是 `options`，当被调用时可以传递给 `apply` 方法。


### `apply`

继续更新进程（只要 `module.hot.status() === 'ready'）。

``` js
module.hot.apply(options, (error, outdatedModules) => {
  // 捕获错误和超时的模块……
})
```

可选的 `options` 对象可以是以下属性：

- `ignoreUnaccepted` (boolean)：即使某些模块不被接受(not accepted)，继续更新进程。


### `addStatusHandler`

注册一个函数来监听 `status`的变化。

``` js
module.hot.addStatusHandler(status => {
  // 响应当前状态……
})
```


### `removeStatusHandler`

移除一个注册的状态处理函数。

``` js
module.hot.removeStatusHandler(callback)
```

***

> 原文：https://webpack.js.org/api/hot-module-replacement/
