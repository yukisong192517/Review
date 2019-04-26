# require

## require的基本用法

`require('./a.js')`

1. node遇到了require(x)的时候，按照下面的顺序进行处理
    * 是否是内置模块
    * 如果以相对路径开头，根据父路径确定绝对路径
    * x可能是文件，如果下面的文件只要存在就返回文件`x,x.js,x.json,x.node`
    * x如果是目录，一次查找下面的文件，如果存在就返回该文件。 `X/package.json,X/index.js,X/index.json,X/index.node`。
    * 如果x 不带路径，确定父模块，推出安装目录。在每个目录中将x当成文件名或者目录名加载。
    * 如果找不到就抛出错误
    
## 源码

### 构造函数

```javascript

function Module(id, parent) {
  this.id = id;
  this.exports = {};
  this.parent = parent;
  this.filename = null;
  this.loaded = false;
  this.children = [];
}

module.exports = Module;

var module = new Module(filename, parent);
```
所以所有的模块都是Module的实例。每个实例都有自己的属性。

### require方法

每个模块实例都有require方法,模块的内部才能使用require命令。

```javascript
Module.prototype.require = function(path) {
  return Module._load(path, this);
};
```

在require内部通过load函数进行模块的加载。首先解析出模块的绝对路径（filename），以它作为模块的识别符。然后，如果模块已经在缓存中，就从缓存取出；如果不在缓存中，就加载模块

```javascript
Module._load = function(request, parent, isMain) {

  //  计算绝对路径
  var filename = Module._resolveFilename(request, parent);

  //  第一步：如果有缓存，取出缓存
  var cachedModule = Module._cache[filename];
  if (cachedModule) {
    return cachedModule.exports;

  // 第二步：是否为内置模块
  if (NativeModule.exists(filename)) {
    return NativeModule.require(filename);
  }

  // 第三步：生成模块实例，存入缓存
  var module = new Module(filename, parent);
  Module._cache[filename] = module;

  // 第四步：加载模块
  try {
    module.load(filename);
    hadException = false;
  } finally {
    if (hadException) {
      delete Module._cache[filename];
    }
  }

  // 第五步：输出模块的exports属性
  return module.exports;
};
```

### 绝对路径的解析

先判断是否是内置模块，然后确定可能的路径进行查找，如果找到就返回，找不到就抛出错误。

```javascript

Module._resolveFilename = function(request, parent) {

  // 第一步：如果是内置模块，不含路径返回
  if (NativeModule.exists(request)) {
    return request;
  }

  // 第二步：确定所有可能的路径
  var resolvedModule = Module._resolveLookupPaths(request, parent); //列出可能的路径
  var id = resolvedModule[0]; 
  var paths = resolvedModule[1];

  // 第三步：确定哪一个路径为真
  var filename = Module._findPath(request, paths);
  if (!filename) {
    var err = new Error("Cannot find module '" + request + "'");
    err.code = 'MODULE_NOT_FOUND';
    throw err;
  }
  return filename;
};
```

### 加载模块

加载模块的时候先确定下来模块的后缀名，不同的后缀采用的不同的加载方式。
以js为例子，首先会通过readfile读文件，然后再进行编译。

编译之后的结果就是
```javascript
(function (exports, require, module, __filename, __dirname) {
  // 模块源码
});
```
通过混入exports、require、module三个全局变量，然后执行模块的源码，然后将模块的 exports 变量的值输出。