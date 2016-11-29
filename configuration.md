
> webpack is fed a configuration object. Depending on your usage of webpack there are two ways to pass this configuration object:

### CLI

If you use the [[CLI]] it will read a file `webpack.config.js` (or the file passed by the `--config` option). This file should export the configuration object:

``` javascript
module.exports = {
	// configuration
};
```

### node.js API

If you use the [[node.js API]] you need to pass the configuration object as parameter:

``` javascript
webpack({
	// configuration
}, callback);
```

### multiple configurations

In both cases you can also use an array of configurations, which are processed in parallel. They share filesystem cache and watchers so this is more efficient than calling webpack multiple times.


# configuration object content

> Hint: Keep in mind that you don't need to write pure JSON into the configuration. Use any JavaScript you want. It's just a node.js module...

Very simple configuration object example:

``` javascript
{
	context: __dirname + "/app",
	entry: "./entry",
	output: {
		path: __dirname + "/dist",
		filename: "bundle.js"
	}
}
```



## `context`

The base directory (absolute path!) for resolving the `entry` option. If `output.pathinfo` is set, the included pathinfo is shortened to this directory.

解析 entry 中配置的根路径，使用绝对路径。如果设置了 output.pathinfo，则 pathinfo 将已此路径计算出相对路径显示。

> Default: `process.cwd()`



## `entry`

The entry point for the bundle.

打包文件的入口点

If you pass a string: The string is resolved to a module which is loaded upon startup.

如果使用字符串：这个字符串将被解析为一个模块，程序将从这个模块开始运行。

If you pass an array: All modules are loaded upon startup. The last one is exported.

如果是一个数组：则数组中的所有模块都会被加载并运行。而最后一个将作为出口文件输出。

``` javascript
entry: ["./entry1", "./entry2"]
```

If you pass an object: Multiple entry bundles are created. The key is the chunk name. The value can be a string or an array.

如果是一个object：则看做是多入口点配置，即多个入口文件，并对应的编译出多个打包文件的配置。object的key将被作为chunk的名字，value的处理过程如上面的两种情况：字符串或数组。

``` javascript
{
	entry: {
		page1: "./page1",
		page2: ["./entry1", "./entry2"]
	},
	output: {
		// Make sure to use [name] or [id] in output.filename
		//  when using multiple entry points
		filename: "[name].bundle.js",
		chunkFilename: "[id].bundle.js"
	}
}
```

> **NOTE**: It is not possible to configure other options specific to entry points. If you need entry point specific configuration you need to use [multiple configurations](#multiple-configurations).



## `output`

Options affecting the output of the compilation. `output` options tell Webpack how to write the compiled files to disk. Note, that while there can be multiple `entry` points, only one `output` configuration is specified.

控制编译输出的选项。`output`选项将告诉Webpack如何将编译后的文件写到磁盘上。注意，如果当前是一个多`entry`，也仅仅只有指定一个`output`配置。

If you use any hashing (`[hash]` or `[chunkhash]`), make sure to have a consistent ordering of modules. Use the `OccurrenceOrderPlugin` or `recordsPath`.

如果你使用任何 hashing（`[hash]` 或 `[chunkhash]`），模块一定要有一个统的次序。使用OccurrenceOrderPlugin 或 recordsPath。

### `output.filename`

Specifies the name of each output file on disk. You must **not** specify an absolute path here! The `output.path` option determines the location on disk the files are written. `filename` is used solely for naming the individual files.

指定每个输出文件的文件名。这个**决不能**设置一个绝对路径。`output.path`才是用来控制在输出的根路径的配置项。`filename`仅用来命名那些文件。

**single entry**
``` javascript
{
  entry: './src/app.js',
  output: {
    filename: 'bundle.js',
    path: __dirname + '/build'
  }
}

// writes to disk: ./build/bundle.js
```

**multiple entries**

If your configuration creates more than a single "chunk" (as with multiple entry points or when using plugins like CommonsChunkPlugin), you should use substitutions to ensure that each file has a unique name.

如果你创建了超过一个 chunk （像有多个入口点，或者使用了类似 CommonsChunkPlugin 这样的插件），你就需要使用**替代符**来保证每个文件有一个唯一的名字。

`[name]` is replaced by the name of the chunk.

`[name]` 用chunk的name替换

`[hash]` is replaced by the hash of the compilation.

`[hash]` 用编译后文件的hash值替换

`[chunkhash]` is replaced by the hash of the chunk.

`[chunkhash]` 用chunk的hash值替换

``` javascript
{
  entry: {
    app: './src/app.js',
    search: './src/search.js'
  },
  output: {
    filename: '[name].js',
    path: __dirname + '/build'
  }
}

// writes to disk: ./build/app.js, ./build/search.js
```

### `output.path`

The output directory as an **absolute path** (required).

输出目录，必须使用**绝对路径**。

`[hash]` is replaced by the hash of the compilation.

`[hash]` 用编译文件的hash替换。

### `output.publicPath`

The `publicPath` specifies the public URL address of the output files when referenced in a browser. For loaders that embed `<script>` or `<link>` tags or reference assets like images, `publicPath` is used as the `href` or `url()` to the file when it's different than their location on disk (as specified by `path`). This can be helpful when you want to host some or all output files on a different domain or on a CDN. The Webpack Dev Server also uses this to determine the path where the output files are expected to be served from. As with `path` you can use the `[hash]` substitution for a better caching profile.

`publicPath`用来指定在浏览器中访问时，输出文件的URL地址的公共部分。对加载器，嵌入到`<script>`或`<link>`标签，或引入的图片文件，当`href`或`url()`中的访问路径和本地路径（通过`path`指定）不同时，需要设置`publicPath`。当你需要将一些或所有文件指向不同域名或CDN时，这个配置项很方便。Webpack Dev Server 同样使用了这个配置去明确输出文件是从那里取到的。同`path`一样，这个配置项也可以使用`[hash]`替换符，去改善缓存策略。

**config.js**

``` javascript
output: {
	path: "/home/proj/public/assets",
	publicPath: "/assets/"
}

```

**index.html**
``` html
<head>
  <link href="/assets/spinner.gif"/>
</head>
```
And a more complicated example of using a CDN and hashes for assets.

**config.js**

``` javascript
output: {
	path: "/home/proj/cdn/assets/[hash]",
	publicPath: "http://cdn.example.com/assets/[hash]/"
}
```

**Note:** In cases when the eventual `publicPath` of output files isn't known at compile time, it can be left blank and set dynamically at runtime in the entry point file. If you don't know the `publicPath` while compiling, you can omit it and set `__webpack_public_path__` on your entry point.

``` javascript
 __webpack_public_path__ = myRuntimePublicPath

// rest of your application entry
```

### `output.chunkFilename`

The filename of non-entry chunks as a relative path inside the `output.path` directory.

没有入口文件的chunk的文件名，是一个基于`output.path`的相对路径。

`[id]` is replaced by the id of the chunk.

`[id]` 用chunk的id替换.

`[name]` is replaced by the name of the chunk (or with the id when the chunk has no name).

`[name]` 用chunk的name替换 (当chunk没有名字的时候，使用id).

`[hash]` is replaced by the hash of the compilation.

`[hash]` 用编译后文件的hash值替换

`[chunkhash]` is replaced by the hash of the chunk.

`[chunkhash]` 用chunk的hash值替换

### `output.sourceMapFilename`

The filename of the SourceMaps for the JavaScript files. They are inside the `output.path` directory.

JavaScript文件的SourceMaps文件名。同样基于`output.path`。

`[file]` is replaced by the filename of the JavaScript file.

`[file]` 用Javascript的文件名替换。

`[id]` is replaced by the id of the chunk.

`[id]` 用chunk的id替换。

`[hash]` is replaced by the hash of the compilation.

`[hash]` 用编译文件的hash替换。

> Default: `"[file].map"`

### `output.devtoolModuleFilenameTemplate`

Filename template string of function for the `sources` array in a generated SourceMap.

在一个生成SourceMap中`sources`数组，用于函数的文件名模板字符串。

`[resource]` is replaced by the path used by Webpack to resolve the file, including the query params to the rightmost loader (if any).

`[resource]` 用webpack解析的文件的路径替换，包含最右边loader的请求参数（如果有的话）

`[resource-path]` is the same as `[resource]` but without the loader query params.

`[resource-path]` 和`[resource]`相同，只不过没有loader请求参数。

`[loaders]` is the list of loaders and params up to the name of the rightmost loader (only explicit loaders).

`[loaders]` 最右边loader的loader和参数的列表（仅显式的loader）

`[all-loaders]` is the list of loaders and params up to the name of the rightmost loader (including automatic loaders).

`[all-loaders]` 最右边loader的loader和参数的列表（包括自动的loader）

`[id]` is replaced by the id of the module.

`[id]` 用module的id替换.

`[hash]` is replaced by the hash of the module identifier.

`[hash]` 用module标识符的hash替换

`[absolute-resource-path]` is replaced with the absolute filename.

`[absolute-resource-path]` 用文件的绝对路径文件名替换。

> Default (devtool=`[inline-]source-map`): `"webpack:///[resource-path]"`  
> Default (devtool=`eval`): `"webpack:///[resource-path]?[loaders]"`  
> Default (devtool=`eval-source-map`): `"webpack:///[resource-path]?[hash]"`

Can also be defined as a function instead of a string template.
The function will accept an `info` object parameter which exposes the following properties:
- identifier
- shortIdentifier
- resource
- resourcePath
- absoluteResourcePath
- allLoaders
- query
- moduleId
- hash

### `output.devtoolFallbackModuleFilenameTemplate`

Similar to `output.devtoolModuleFilenameTemplate` but used in the case of duplicate module identifiers.

> Default: `"webpack:///[resourcePath]?[hash]"`

### `output.devtoolLineToLine`

Enable line-to-line mapped mode for all/specified modules. Line-to-line mapped mode uses a simple SourceMap where each line of the generated source is mapped to the same line of the original source. It's a performance optimization. Only use it if your performance needs to be better and you are sure that input lines match which generated lines.

`true` enables it for all modules (not recommended)

An object `{test, include, exclude}` similar to `module.loaders` enables it for specific files.

> Default: disabled

### `output.hotUpdateChunkFilename`

The filename of the Hot Update Chunks. They are inside the `output.path` directory.

热加载chunk的文件名。基于`output.path`。

`[id]` is replaced by the id of the chunk.

`[id]` 用chunk的id替换

`[hash]` is replaced by the hash of the compilation. (The last hash stored in the records)

`[hash]` 用编译文件的hash替换。（存储在记录中的最后的hash）

> Default: `"[id].[hash].hot-update.js"`

### `output.hotUpdateMainFilename`

The filename of the Hot Update Main File. It is inside the `output.path` directory.

热加载主文件的文件名。基于`output.path`。

`[hash]` is replaced by the hash of the compilation. (The last hash stored in the records)

`[hash]` 用编译文件的hash替换。（存储在记录中的最后的hash）

> Default: `"[hash].hot-update.json"`

### `output.jsonpFunction`

The JSONP function used by webpack for async loading of chunks.

使用webpack异步加载chunk的JSONP函数名。

A shorter function may reduce the filesize a bit. Use a different identifier when having multiple webpack instances on a single page.

一个尽量短的名字可以很有效的减少文件大小。当有多个webpack实例运行在一个页面时，使用一个不同的标识符。

> Default: `"webpackJsonp"`

### `output.hotUpdateFunction`

The JSONP function used by webpack for async loading of hot update chunks.

使用webpack异步热加载chunk时的JSONP函数名。

> Default: `"webpackHotUpdate"`

### `output.pathinfo`

Include comments with information about the modules.

引入有关模块的信息评论。

`require(/* ./test */23)`

Do not use this in production.

不能在生产环境使用

> Default: `false`

### `output.library`

If set, export the bundle as library. `output.library` is the name.

如果设置此项，将打包文件当做一个类库输出。`output.library`为类库的名字

Use this if you are writing a library and want to publish it as single file.

当你需要编写一个类库，同时需要发布为一个独立文件时使用。

### `output.libraryTarget`

Which format to export the library:

类库文件适用于那种规范。

`"var"` - Export by setting a variable: `var Library = xxx` (default)

`"var"` - 按设置变量的方式输出: `var Library = xxx` (default)

`"this"` - Export by setting a property of `this`: `this["Library"] = xxx`

`"this"` - 按设置`this`属性的方式输出: `this["Library"] = xxx`

`"commonjs"` - Export by setting a property of `exports`: `exports["Library"] = xxx`

`"commonjs"` - 按设置`exports`属性的方式输出: `exports["Library"] = xxx`

`"commonjs2"` - Export by setting `module.exports`: `module.exports = xxx`

`"commonjs2"` - 按设置`module.exports`的方式输出: `module.exports = xxx`

`"amd"` - Export to AMD (optionally named - set the name via the library option)

`"amd"` - 使用AMD规范输出(使用library设置的名字命名模块)

`"umd"` - Export to AMD, CommonJS2 or as property in root

`"umd"` - 使用兼容AMD，CommonJS2，或作为root一个属性的方式输出。

> Default: `"var"`

If `output.library` is not set, but `output.libraryTarget` is set to a value other than `var`, every property of the exported object is copied (Except `amd`, `commonjs2` and `umd`).

如果`output.library`没有被设置，但`output.libraryTarget`被设置了一个不是`var`的值，要输出对象的所有的属性都将备份一份（`amd`, `commonjs2` and `umd`除外）。

### `output.umdNamedDefine`

If `output.libraryTarget` is set to `umd` and `output.library` is set, setting this to `true` will name the AMD module.

如果`output.libraryTarget` 设置为`umd`，同时也设置了`output.library`, 设置此项为`true`，将命名AMD模块。

### `output.sourcePrefix`

Prefixes every line of the source in the bundle with this string.

为打包的每行添加这个字符串。

> Default: `"\t"`

### `output.crossOriginLoading`

This option enables cross-origin loading of chunks.

这个配置项控制打开跨域加载chunk

Possible values are:

允许的值有：

`false` - Disable cross-origin loading.

`false` - 不允许跨域加载

`"anonymous"` - Cross-origin loading is enabled. When using `anonymous` no credentials will be sent with the request.

`"anonymous"` - 允许跨域加载. 当使用`anonymous`，不会有资格认证随着request发送。

`"use-credentials"` - Cross-origin loading is enabled and credentials will be send with the request.

`"use-credentials"` - 允许跨域加载，同时有资格认证随request发送。

For more information on cross-origin loading see [MDN](https://developer.mozilla.org/en/docs/Web/HTML/Element/script#attr-crossorigin)

> Default: `false`



## `module`

Options affecting the normal modules (`NormalModuleFactory`)

控制normal modules的配置项 (`NormalModuleFactory`)

### `module.loaders`

An array of automatically applied loaders.

自动运行应用的loader数组

Each item can have these properties:

数组每项包含如下属性：

* `test`: A condition that must be met
* `test`: 必须匹配的条件
* `exclude`: A condition that must not be met
* `exclude`: 必须排除的条件
* `include`: An array of paths or files where the imported files will be transformed by the loader
* `include`: 路径或文件的数组，在这个数组里面的文件将被loader编译转换。
* `loader`: A string of "!" separated loaders
* `loader`: 使用"!"分隔的loader
* `loaders`: An array of loaders as string
* `loaders`: loader的字符串数组。

A condition may be a `RegExp` (tested against absolute path), a `string` containing the absolute path, a `function(absPath): bool`, or an array of one of these combined with "and".

匹配添加可以是`RegExp` (测试目标的绝对路径)，一个`string`，含有绝对路径， 一个`function(absPath): bool`，或者上述之一的一个用『and』拼接的数组。

See more: [[loaders]]

*IMPORTANT*: The loaders here are resolved *relative to the resource* which they are applied to. This means they are not resolved relative to the configuration file. If you have loaders installed from npm and your `node_modules` folder is not in a parent folder of all source files, webpack cannot find the loader. You need to add the `node_modules` folder as an absolute path to the `resolveLoader.root` option. (`resolveLoader: { root: path.join(__dirname, "node_modules") }`)

*重要*: 这里将要解析的loader，相对于那些要应用的resource。这意味着解析时并不是相对于配置文件的。如果你通过npm安装的loader，同时你的`node_modules`不在所有源码文件的父级目录，webpack将找不到loader。你需要将`node_modules`目录的绝对路径添加到`resolveLoader.root`配置项中。(`resolveLoader: { root: path.join(__dirname, "node_modules") }`)

Example:

``` javascript
module: {
  loaders: [
    {
      // "test" is commonly used to match the file extension
      // "test" 经常用来匹配文件的后缀
      test: /\.jsx$/,

      // "include" is commonly used to match the directories
      // "include" 经常用来匹配目录
      include: [
        path.resolve(__dirname, "app/src"),
        path.resolve(__dirname, "app/test")
      ],

      // "exclude" should be used to exclude exceptions
      // "exclude" 应该被用来排除例外。
      // try to prefer "include" when possible
      // 如果可以尽可能使用"include"

      // the "loader"
      loader: "babel-loader" // or "babel" because webpack adds the '-loader' automatically 或者 "bable"，因为webpack会自动添加'-loader'
    }
  ]
}
```

### `module.preLoaders`, `module.postLoaders`

Syntax like `module.loaders`.

语法与`module.loaders`相同。

An array of applied pre and post loaders.

一个应用于pre（预）和post（提交）的loader的数组。

### `module.noParse`

Don't parse files matching a RegExp or an array of RegExps.

用于匹配不编译文件的一个正则或一个正则数组。

It's matched against the full resolved request.

它会匹配所有的解析请求。

This can boost the performance when ignoring big libraries.

当忽略一些大的类库时，可以有效的提高效率。

The files are expected to have no call to `require`, `define`, or similar. They are allowed to use `exports` and `module.exports`.

这些文件中不能调用`require`，`define`，或类似的方法。允许使用`exports`和`module.exports`。

### automatically created contexts defaults `module.xxxContextXxx` 自动创建上下文默认值 `module.xxxContextXxx`

There are multiple options to configure the defaults for an automatically created context. We differentiate three types of automatically created contexts:

这是一个复数配置项，用于配置那些自动创建上下文的默认值。我们区分三种自动创建上下文。

* `exprContext`: An expression as a dependency (i. e. `require(expr)`)
* `exprContext`: 一个依赖表达式(i. e. `require(expr)`)
* `wrappedContext`: An expression plus pre- and/or suffixed string (i. e. `require("./templates/" + expr)`)
* `wrappedContext`: 前缀后后缀表达式字符串(i. e. `require("./templates/" + expr)`)
* `unknownContext`: Any other unparsable usage of `require` (i. e. `require`)
* `unknownContext`: 其他一些不能解析的`require` (i. e. `require`)

Four options are possible for automatically created contexts:

自动创建上下文允许下面四种配置值：

* `request`: The request for context.
* `request`: 上下文的请求.
* `recursive`: Subdirectories should be traversed.
* `recursive`: 子目录需要被穿越
* `regExp`: The RegExp for the expression.
* `regExp`: RegExp表达式
* `critical`: This type of dependency should be consider as critical (emits a warning).
* `critical`: 这个种类的依赖应当被慎重考虑(emits a warning).

All options and defaults:
所有配置项的默认值:

`unknownContextRequest = "."`, `unknownContextRecursive = true`, `unknownContextRegExp = /^\.\/.*$/`, `unknownContextCritical = true`

`exprContextRequest = "."`, `exprContextRegExp = /^\.\/.*$/`, `exprContextRecursive = true`, `exprContextCritical = true`

`wrappedContextRegExp = /.*/`, `wrappedContextRecursive = true`, `wrappedContextCritical = false`

> Note: `module.wrappedContextRegExp` only refers to the middle part of the full RegExp. The remaining is generated from prefix and suffix.

> 注：`module.wrappedContextRegExp`仅指的是全部正则的中间部分。剩余部分由prefix和suffix生成。

Example:

``` javascript
{
  module: {
	// Disable handling of unknown requires
  // 关闭处理不知道的require
	unknownContextRegExp: /$^/,
	unknownContextCritical: false,

	// Disable handling of requires with a single expression
  // 关闭处理只有一个表达式的require
	exprContextRegExp: /$^/,
	exprContextCritical: false,

	// Warn for every expression in require
  // 输出警告所有的require中的表达式
	wrappedContextCritical: true
  }
}
```



## `resolve`

Options affecting the resolving of modules.

影响解析modules的配置项。

### `resolve.alias`

Replace modules with other modules or paths.

用另一个module或路径替换module。

Expects an object with keys being module names. The value is the new path. It's similar to a replace but a bit more clever. If the key ends with `$` only the exact match (without the `$`) will be replaced.

要求使用一个含有module名作为键的对象。它值是新的路径。有点像替换当要更智能一点。如果键用`$`结尾，则精确匹配（去掉`$`）替换。

If the value is a relative path it will be relative to the file containing the require.

如果值是相对路径，它将相对于调用require的文件的路径。

Examples: Calling a require from `/abc/entry.js` with different alias settings.

Examples: 文件`/abc/entry.js`调用require了，在不同的别名设置下的情况。


| `alias:` | `require("xyz")` | `require("xyz/file.js")` |
|---|---|---|
| `{}` | `/abc/node_modules/xyz/index.js` | `/abc/node_modules/xyz/file.js` |
| `{ xyz: "/absolute/path/to/file.js" }` | `/absolute/path/to/file.js` | error |
| `{ xyz$: "/absolute/path/to/file.js" }` | `/absolute/path/to/file.js` | `/abc/node_modules/xyz/file.js` |
| `{ xyz: "./dir/file.js" }` | `/abc/dir/file.js` | error |
| `{ xyz$: "./dir/file.js" }` | `/abc/dir/file.js` | `/abc/node_modules/xyz/file.js` |
| `{ xyz: "/some/dir" }` | `/some/dir/index.js` | `/some/dir/file.js` |
| `{ xyz$: "/some/dir" }` | `/some/dir/index.js` | `/abc/node_modules/xyz/file.js` |
| `{ xyz: "./dir" }` | `/abc/dir/index.js` | `/abc/dir/file.js` |
| `{ xyz: "modu" }` | `/abc/node_modules/modu/index.js` | `/abc/node_modules/modu/file.js` |
| `{ xyz$: "modu" }` | `/abc/node_modules/modu/index.js` | `/abc/node_modules/xyz/file.js` |
| `{ xyz: "modu/some/file.js" }` | `/abc/node_modules/modu/some/file.js` | error |
| `{ xyz: "modu/dir" }` | `/abc/node_modules/modu/dir/index.js` | `/abc/node_modules/dir/file.js` |
| `{ xyz: "xyz/dir" }` | `/abc/node_modules/xyz/dir/index.js` | `/abc/node_modules/xyz/dir/file.js` |
| `{ xyz$: "xyz/dir" }` | `/abc/node_modules/xyz/dir/index.js` | `/abc/node_modules/xyz/file.js` |

`index.js` may resolve to another file if defined in the `package.json`.

如果配置了`package.json`，`index.js`有可能解析为另一个文件

`/abc/node_modules` may resolve in `/node_modules` too.

`/abc/node_modules`也可能解析为`/node_modules`。

### `resolve.root`

The directory (**absolute path**) that contains your modules. May also be an array of directories. This setting should be used to add individual directories to the search path.

包含你的module的目录（**绝对路径**）。同样也可以为一组目录的数组。此设置应用于将单个目录添加到搜索路径。

> It **must** be an **absolute path**! Don't pass something like `./app/modules`.

> 这里**必须**是一个**绝对路径**！不用使用这种`./app/modules`

Example:

``` javascript
var path = require('path');

// ...
resolve: {
  root: [
    path.resolve('./app/modules'),
    path.resolve('./vendor/modules')
  ]
}
```

### `resolve.modulesDirectories`

An array of directory names to be resolved to the current directory as well as its ancestors, and searched for modules. This functions similarly to how node finds "node_modules" directories. For example, if the value is `["mydir"]`, webpack will look in "./mydir", "../mydir", "../../mydir", etc.

一个目录名数组，那些要解析为当前目录，和搜索模块的目录。这些方法有点像node如何在"node_modules"目录下查询。例如，如果值为`["mydir"]`，webpack将查询"./mydir", "../mydir", "../../mydir", 等等。

> Default: `["web_modules", "node_modules"]`

> Note: Passing `"../someDir"`, `"app"`, `"."` or an absolute path isn't necessary here. Just use a directory name, not a path. Use only if you expect to have a hierarchy within these folders. Otherwise you may want to use the `resolve.root` option instead.

> 注：这里不需要`"../someDir"`、`"app"`、`"."`或一个绝对路径。使用目录名即可，不需要路径。仅在你预计有一个多层级的目录结构时使用。其他情况下用`resolve.root`代替。

### `resolve.fallback`

A directory (or array of directories **absolute paths**), in which webpack should look for modules that weren't found in `resolve.root` or `resolve.modulesDirectories`.

一个目录（或目录数组**绝对路径**），webpack在`resolve.root`或`resolve.modulesDirectories`中没有找到module时，去这里查找。

### `resolve.extensions`

An array of extensions that should be used to resolve modules. For example, in order to discover CoffeeScript files, your array should contain the string `".coffee"`.

一个后缀名的数组，解析module时使用。例如，需要查找CoffeeScript文件时，你的数组应当包含`".coffee"`。

> Default: `[".webpack.js", ".web.js", ".js"]`

**IMPORTANT**: Setting this option will override the default, meaning that webpack will no longer try to resolve modules using the default extensions. If you want modules that were required with their extension (e.g. `require('./somefile.ext')`) to be properly resolved, you **must** include an empty string in your array. Similarly, if you want modules that were required without extensions (e.g. `require('underscore')`) to be resolved to files with ".js" extensions, you **must** include `".js"` in your array.

**重要**：设置这个配置项将覆盖默认值，意味着webpack不会使用默认的后缀名去解析module。如果你想要module在require的时候使用后缀(e.g. `require('./somefile.ext')`)时被正常的解析，你**必须**添加一个空的字符串在这个数组中。同样的，如果你希望require的时候不是要后缀，而能正确解析为".js"文件，则你**必须**将`".js"`添加到你的数组中。

### `resolve.packageMains`

Check these fields in the `package.json` for suitable files.

在`package.json`中查找符合这些字段的文件。

> Default: `["webpack", "browser", "web", "browserify", ["jam", "main"], "main"]`

**Note**: This option has been changed to `resolve.mainFields` in webpack 2.

**注**：这个配置项在webpack2中改为`resolve.mainFields`。

### `resolve.packageAlias`

Check this field in the `package.json` for an object. Key-value-pairs are treated as aliasing according to [this spec](https://github.com/defunctzombie/package-browser-field-spec)

在`package.json`中查询对象里的字段，键值对是按照这个规范的别名来进行的[this spec](https://github.com/defunctzombie/package-browser-field-spec)

> Not set by default

Example: `"browser"` to check the browser field.

### `resolve.unsafeCache`

Enable aggressive but unsafe caching for the resolving of a part of your files. Changes to cached paths may cause failure (in rare cases). An array of RegExps, only a RegExp or `true` (all files) is expected. If the resolved path matches, it'll be cached.

启用不安全缓存来解析一部分文件。改变缓存路径可能引发报错（比较罕见的情况下）。一个正则数组，其中只能设置正则或`true`。如果解析路径与之匹配，将被缓存。

> Default: `[]`



## `resolveLoader`

Like `resolve` but for loaders.

与`resolve`，但用来控制loader

``` javascript
// Default:
{
	modulesDirectories: ["web_loaders", "web_modules", "node_loaders", "node_modules"],
	extensions: ["", ".webpack-loader.js", ".web-loader.js", ".loader.js", ".js"],
	packageMains: ["webpackLoader", "webLoader", "loader", "main"]
}
```

Note that you can use `alias` here and other features familiar from `resolve`. For example `{ txt: 'raw-loader' }` would shim `txt!templates/demo.txt` to use `raw-loader`.

注意，这里同样可以使用`alias`，其他特性和`resolve`相似。例如在使用`raw-loader`时，配置`{ txt: 'raw-loader' }`，可以简化为`txt!templates/demo.txt`。

### `resolveLoader.moduleTemplates`

That's a `resolveLoader` only property.

这是`resolveLoader`独有的属性。

It describes alternatives for the module name that are tried.

它描述了那些用来尝试的模块名称的替代名。

> Default: `["*-webpack-loader", "*-web-loader", "*-loader", "*"]`



## `externals`

Specify dependencies that shouldn't be resolved by webpack, but should become dependencies of the resulting bundle. The kind of the dependency depends on `output.libraryTarget`.

指定那些依赖不需要被webpack解析，但会成为bundle（打包文件）的依赖。由`output.libraryTarget`决定依赖适用的规范。

As value an object, a string, a function, a RegExp, and an array is accepted.

对象，字符串，函数，正则，数组都可以作为值。

* string: An exact matched dependency becomes external. The same string is used as external dependency.
* string: 精确匹配一个依赖，将其设置为外部依赖。在外部调用时使用同样的字符串。
* object: If an dependency matches exactly a property of the object, the property value is used as dependency. The property value may contain a dependency type prefixed and separated with a space. If the property value is `true` the property name is used instead. If the property value is `false` the externals test is aborted and the dependency is not external. See example below.
* object: 如果依赖精确匹配到了一个对象的属性，这个属性的值将作为依赖。属性的值可以包括一个依赖类型的前缀，使用空格分隔。如果属性的值为`true`，则使用该属性名。如果为`false`，则放弃外部测试，这个依赖不是外部依赖。
* function: `function(context, request, callback(err, result))` The function is called on each dependency. If a result is passed to the callback function this value is handled like a property value of an object (above bullet point).
* function: `function(context, request, callback(err, result))` 函数会在每个依赖中调用。如果结果被传递到回调函数里，这个值就会被像处理对象属性值那样处理。
* RegExp: Every matched dependency becomes external. The matched text is used as the `request` for the external dependency.  Because the `request` _is the exact code_ used to generate the external code hook, if you are matching a commonjs package (e.g. '../some/package.js'), instead use the function external strategy. You can import the package via `callback(null, "require('" + request + "')"`, which generates a `module.exports = require('../some/package.js');`, using require outside of webpack context.
* RegExp: 每个被匹配的依赖都会成为外部依赖。匹配的文本会被用作外部依赖的`request`。因为`request`是用于生成外部代码钩子的确切代码，如果你匹配到一个cmd的包(比如 ‘../some/package.js’)，相反使用外部function的策略。你可以通过callback(null, "require('" + request + "')"引入包，这个包生成module.exports = require('../some/package.js')；在webpack上下文使用require外。
* array: Multiple values of the scheme (recursive).
* array: 多个值的表 (递归)。

Example:

``` javascript
{
	output: { libraryTarget: "commonjs" },
	externals: [
		{
			a: false, // a is not external a不是外部依赖
			b: true, // b is external (require("b")) b是外部依赖，使用 require("b") 调用。
			"./c": "c", // "./c" is external (require("c")) "./c"是外部依赖，使用 require("c") 调用。
			"./d": "var d" // "./d" is external (d) "./d"是外部依赖，直接使用 d 调用。
		},
		// Every non-relative module is external
    // 所以非相对路径的module都是外部依赖
		// abc -> require("abc")
		/^[a-z\-0-9]+$/,
		function(context, request, callback) {
			// Every module prefixed with "global-" becomes external
			// "global-abc" -> abc
			if(/^global-/.test(request))
				return callback(null, "var " + request.substr(7));
			callback();
		},
		"./e" // "./e" is external (require("./e")) "./e"是外部依赖，使用 require("e") 调用。
	]
}
```

| type        | value               | resulting import code |
|-------------|---------------------|-----------------------|
| "var"       | `"abc"`             | `module.exports = abc;` |
| "var"       | `"abc.def"`         | `module.exports = abc.def;` |
| "this"      | `"abc"`             | `(function() { module.exports = this["abc"]; }());` |
| "this"      | `["abc", "def"]`    | `(function() { module.exports = this["abc"]["def"]; }());` |
| "commonjs"  | `"abc"`             | `module.exports = require("abc");` |
| "commonjs"  | `["abc", "def"]`    | `module.exports = require("abc").def;` |
| "amd"       | `"abc"`             | `define(["abc"], function(X) { module.exports = X; })` |
| "umd"       | `"abc"`             | everything above |

Enforcing `amd` or `umd` in a external value will break if not compiling as amd/umd target.

如果没有解析为amd/umd模块，在外部值中强制执行`amd`或`umd`将会中断。

> Note: If using `umd` you can specify an object as external value with property `commonjs`, `commonjs2`, `amd` and `root` to set different values for each import kind.

> 注：如果使用`umd`规范，你可以指定一个对象的外部调用值，对不同的import类型`commonjs`、`commonjs2`、`amd`和`root`设置不同的属性值。




## `target`

* `"web"` Compile for usage in a browser-like environment (default)
* `"web"` 浏览器环境 (默认值)
* `"webworker"` Compile as WebWorker
* `"webworker"` WebWorker环境
* `"node"` Compile for usage in a node.js-like environment (use `require` to load chunks)
* `"node"` node.js或类node.js环境 (使用`require`加载chunk)
* `"async-node"` Compile for usage in a node.js-like environment (use `fs` and `vm` to load chunks async)
* `"async-node"` node.js或类node.js环境 (使用 `fs` 和 `vm` 异步加载chunk)
* `"node-webkit"` Compile for usage in webkit, uses jsonp chunk loading but also supports build in node.js modules plus require("nw.gui") (experimental)
* `"node-webkit"` webkit环境, 使用jsonp加载chunk，同样支持在node.js的模块插件require("nw.gui")中构建(实验性质)。
* `"electron"` Compile for usage in [Electron](http://electron.atom.io/) – supports `require`-ing Electron-specific modules.
* `"electron"` 在[Electron](http://electron.atom.io/)环境 – 支持 `require`-ing Electron-specific modules.
* `"electron-renderer"` Compile for electron renderer process, provide a target using `JsonpTemplatePlugin`, `FunctionModulePlugin` for browser environment and `NodeTargetPlugin` and `ExternalsPlugin` for commonjs and electron bulit-in modules. *Note: need `webpack` >= 1.12.15.*
* `"electron-renderer"` electron 渲染进程环境， 规定target在浏览器环境使用 `JsonpTemplatePlugin`、`FunctionModulePlugin`，在cmd和electron环境使用`NodeTargetPlugin` and `ExternalsPlugin`。*注: 需要 `webpack` >= 1.12.15.*



## `bail`

Report the first error as a hard error instead of tolerating it.

报告第一个错误，不能忽略。



## `profile`

Capture timing information for each module.

捕获每个模块的时序信息。

> Hint: Use the [analyze tool](http://webpack.github.io/analyse) to visualize it. `--json` or `stats.toJson()` will give you the stats as JSON.

> 提示：使用工具[analyze tool](http://webpack.github.io/analyse)让它可视化。使用`--json`或`stats.toJson()`将分析数据输出为一个JSON。


## `cache`

Cache generated modules and chunks to improve performance for multiple incremental builds.

缓存生成的module和chunk，可以提升复数增量构建的性能。

This is enabled by default in watch mode.

在观察模式中默认开启。

You can pass `false` to disable it.

你可以传递一个`false`禁用它

You can pass an object to enable it and let webpack use the passed object as cache. This way you can share the cache object between multiple compiler calls. Note: Don't share the cache between calls with different options.

你可以传递一个对象启用它，并让webpack把这个传递值作为缓存。用这种方向可以在多个编译器调用中共享缓存。注意：不要在拥有不同配置的调用中共享缓存。


## `debug`

Switch loaders to debug mode.

切换loader到debug模式。


## `devtool`

Choose a developer tool to enhance debugging.

选择一个开发工具来提高调试效率。

`eval` - Each module is executed with `eval` and `//@ sourceURL`.

`eval` - 每个module使用`eval`和`//@ sourceURL`执行。

`source-map` - A SourceMap is emitted. See also `output.sourceMapFilename`.

`source-map` - 使用SourceMap，见`output.sourceMapFilename`.

`hidden-source-map` - Same as `source-map`, but doesn't add a reference comment to the bundle.

`hidden-source-map` - 同`source-map`，但不给bundle添加引用。

`inline-source-map` - A SourceMap is added as DataUrl to the JavaScript file.

`inline-source-map` - 使用DataUrl的方式将SourceMap添加进JavaScript文件中。

`eval-source-map` - Each module is executed with `eval` and a SourceMap is added as DataUrl to the `eval`.

`eval-source-map` - 每个module使用`eval`执行，同时将SourceMap添加进`eval`中。

`cheap-source-map` - A SourceMap without column-mappings. SourceMaps from loaders are not used.

`cheap-source-map` - 使用没有column-mappings的SourceMap。不使用由loader创建的SourceMaps。

`cheap-module-source-map` - A SourceMap without column-mappings. SourceMaps from loaders are simplified to a single mapping per line.

`cheap-module-source-map` - 使用没有column-mappings的SourceMap。由loader创建的SourceMaps简化为一行。

Prefixing `@`, `#` or `#@` will enforce a pragma style. (Defaults to `@` in `webpack@1` and `#` in `webpack@2`; using `#` is recommended)

前缀`@`、`#`或`#@`将强制使用[杂注](https://msdn.microsoft.com/zh-cn/library/f88y8c6c.aspx)风格。(在`webpack@1`默认使用`@`，在`webpack@2`默认使用`#`； 推荐使用`#`) 注：“杂注”指示编译器在编译时执行特定操作

Combinations are possible. `hidden`, `inline`, `eval` and pragma style are exclusive.

可以使用混合体。`hidden`、`inline`、`eval`和杂注风格都可以接受。

i. e. `cheap-module-inline-source-map`, `cheap-eval-source-map`, `#@source-map`

> Hint: If your modules already contain SourceMaps you'll need to use the [source-map-loader](https://github.com/webpack/source-map-loader) to merge it with the emitted SourceMap.

> 提示：如果你的module已经有了SourceMap，你需要使用[source-map-loader](https://github.com/webpack/source-map-loader)去与生成的的SourceMap合并。

| devtool                      | build speed | rebuild speed | production supported | quality                 |
|------------------------------|-------------|---------------|----------------------|-------------------------|
| eval                         |     +++     |      +++      |       no       | generated code                |
| cheap-eval-source-map        |      +      |      ++       |       no       | transformed code (lines only) |
| cheap-source-map             |      +      |       o       |       yes      | transformed code (lines only) |
| cheap-module-eval-source-map |      o      |      ++       |       no       | original source (lines only)  |
| cheap-module-source-map      |      o      |       -       |       yes      | original source (lines only)  |
| eval-source-map              |     --      |       +       |       no       | original source               |
| source-map                   |     --      |       --      |       yes      | original source               |

Example:

``` javascript
{
	devtool: "#inline-source-map"
}
// =>
//# sourceMappingURL=...
```

> Note: With the next major version the default for `-d` will change to `cheap-module-eval-source-map`

> 注：在下一个重要的版本中 `-d` 的默认值将会改为 `cheap-module-eval-source-map`

## `devServer`

Can be used to configure the behaviour of [webpack-dev-server](https://github.com/webpack/webpack-dev-server) when the webpack config is passed to webpack-dev-server CLI.

当webpack的配置已经传入了webpack-dev-server命令行工具时，可以用来配置[webpack-dev-server](https://github.com/webpack/webpack-dev-server)的行为。


Example:

``` javascript
{
	devServer: {
		contentBase: "./build",
	}
}
```

## `node`

Include polyfills or mocks for various node stuff:

在node下引入polyfills或mocks。

* `console`: `true` or `false`
* `global`: `true` or `false`
* `process`: `true`, `"mock"` or `false`
* `Buffer`: `true` or `false`
* `__filename`: `true` (real filename relative to the context option), `"mock"` (`"/index.js"`) or `false` (normal node __dirname)
* `__dirname`: `true` (real dirname relative to the context option), `"mock"` (`"/"`) or `false` (normal node __dirname)
* `<node buildin>`: `true`, `"mock"`, `"empty"` or `false`


``` javascript
// Default:
{
	console: false,
	global: true,
	process: true,
	Buffer: true,
	__filename: "mock",
	__dirname: "mock",
	setImmediate: true
}
```


## `amd`

Set the value of `require.amd` and `define.amd`.

设置`require.amd`和`define.amd`的值。

Example: `amd: { jQuery: true }` (for old 1.x AMD versions of jquery)



## `loader`

Custom values available in the loader context.

在loader上下文中可用自定义值。



## `recordsPath`, `recordsInputPath`, `recordsOutputPath`

Store/Load compiler state from/to a json file. This will result in persistent IDs of modules and chunks.

存储（加载）编译器的状态从（到）json文件中。这将持久化module和chunk的ID。

An **absolute path** is expected. `recordsPath` is used for `recordsInputPath` and `recordsOutputPath` if they left undefined.

必须为**绝对路径**。如果`recordsInputPath` and `recordsOutputPath`是undefined的话，`recordsPath`会被启用。

This is required when using Hot Code Replacement between multiple calls to the compiler.

当在多个调用编译器之间使用Hot Code Replacement的时候，这个选项是必须的。


## `plugins`

Add additional plugins to the compiler.

向编译器添加额外的插件。