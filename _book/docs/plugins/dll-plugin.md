---
title: DllPlugin
contributors:
  - aretecode
  - sokra
  - opiepj
  - simon04
  - skipjack
related:
  - title: Code Splitting Example
    url: https://github.com/webpack/webpack/tree/master/examples/explicit-vendor-chunk/README.md
---

The `DllPlugin` and `DllReferencePlugin` provide means to split bundles in a way that can drastically improve build time performance.


## `DllPlugin`

This plugin is used in a separate webpack config exclusively to create a dll-only-bundle. It creates a `manifest.json` file, which is used by the [`DllReferencePlugin`](/plugins/dll-plugin#dllreferenceplugin) to map dependencies.

* `context` (optional): context of requests in the manifest file (defaults to the webpack context.)
* `name`: name of the exposed dll function ([TemplatePaths](https://github.com/webpack/webpack/blob/master/lib/TemplatedPathPlugin.js): `[hash]` & `[name]` )
* `path`: **absolute path** to the manifest json file (output)

```javascript
new webpack.DllPlugin(options)
```

Creates a `manifest.json` which is written to the given `path`. It contains mappings from require and import requests, to module ids. It is used by the `DllReferencePlugin`.

Combine this plugin with [`output.library`](/configuration/output/#output-library) option to expose (aka, put into the global scope) the dll function.


## `DllReferencePlugin`

This plugin is used in the primary webpack config, it references the dll-only-bundle(s) to require pre-built dependencies.

* `context`: (**absolute path**) context of requests in the manifest (or content property)
* `manifest` (object): an object containing `content` and `name`
* `content` (optional): the mappings from request to module id (defaults to `manifest.content`)
* `name` (optional): the name where the dll is exposed (defaults to `manifest.name`) (see also [`externals`](/configuration/externals/))
* `scope` (optional): prefix which is used for accessing the content of the dll
* `sourceType` (optional): how the dll is exposed ([libraryTarget](/configuration/output/#output-librarytarget))

```javascript
new webpack.DllReferencePlugin(options)
```

References a dll manifest file to map dependency names to module ids, then requires them as needed using the internal `__webpack_require__` function.

W> Keep the `name` consistent with [`output.library`](/configuration/output/#output-library).


### Modes

This plugin can be used in two different modes, _scoped_ and _mapped_.

#### Scoped Mode

The content of the dll is accessible under a module prefix. i.e. with `scope = "xyz"` a file `abc` in the dll can be access via `require("xyz/abc")`.

T> [See an example use of scope](https://github.com/webpack/webpack/tree/master/examples/dll-user)

#### Mapped Mode

The content of the dll is mapped to the current directory. If a required file matches a file in the dll (after resolving), then the file from the dll is used instead.

Because this happens after resolving every file in the dll bundle, the same paths must be available for the consumer of the dll bundle. i.e. if the dll contains `lodash` and the file `abc`, `require("lodash")` and `require("./abc")` will be used from the dll, rather than building them into the main bundle.


## Usage

W> `DllReferencePlugin` and `DllPlugin` are used in _separate_ webpack configs.

**webpack.vendor.config.js**

```javascript
new webpack.DllPlugin({
  context: __dirname,
  name: "[name]_[hash]",
  path: path.join(__dirname, "manifest.json"),
})
```

**webpack.app.config.js**

```javascript
new webpack.DllReferencePlugin({
  context: __dirname,
  manifest: require("./manifest.json"),
  name: "./my-dll.js",
  scope: "xyz",
  sourceType: "commonsjs2"
})
```


## Examples

[Vendor](https://github.com/webpack/webpack/tree/master/examples/dll) and [User](https://github.com/webpack/webpack/tree/master/examples/dll-user)

_Two separate example folders. Demonstrates scope and context._

T> Multiple `DllPlugins` and multiple `DllReferencePlugins`.


## References

### Source

* [DllPlugin source](https://github.com/webpack/webpack/tree/master/lib/DllPlugin.js)
* [DllReferencePlugin source](https://github.com/webpack/webpack/tree/master/lib/DllReferencePlugin.js)
* [DllEntryPlugin source](https://github.com/webpack/webpack/blob/master/lib/DllEntryPlugin.js)
* [DllModuleFactory source](https://github.com/webpack/webpack/blob/master/lib/DllModuleFactory.js)
* [ManifestPlugin source](https://github.com/webpack/webpack/blob/master/lib/LibManifestPlugin.js)

### Tests

* [DllPlugin creation test](https://github.com/webpack/webpack/tree/master/test/configCases/dll-plugin/0-create-dll/webpack.config.js)
* [DllPlugin without scope test](https://github.com/webpack/webpack/tree/master/test/configCases/dll-plugin/2-use-dll-without-scope/webpack.config.js)
* [DllReferencePlugin use Dll test](https://github.com/webpack/webpack/tree/master/test/configCases/dll-plugin)
