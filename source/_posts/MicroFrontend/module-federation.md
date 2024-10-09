---
title: 模块联邦 远程载入
tags: [微前端,MicroFrontend,前端全栈]
slug: ea539f7c
keywords: 微前端,前端微服务化,前端自动化,解决方案,前端难题,Single,SPA
date: 2024-06-15 19:25:25
---

# 前言
当项目A需要共享项目B的某一个组件，并需要保证后续的迭代中，两边保持一致。这个时候有两种做法：

CV 大法，将项目B的组件完整复制到项目A
将该组件独立，发布到内部npm，通过npm加载组件

CV 大法比独立组件更快，不需要将组件从项目B独立出来，发布npm，但不能及时同步代码。
当两个项目都使用了Webpack5，通过联邦模块只需要几行配置就可以实现在项目A同步项目B的组件。

## 一.什么是联邦模块（Module Federation）

> 联邦模块是webpack5提供的一个新特性，它是通过webpack原生提供的 `ModuleFederationPlugin` 插件来实现的。
> 联邦模块主要是用来解决多个应用之间代码共享的问题，可以让我们的更加优雅的实现跨应用的代码共享

联邦模块出现的动机通常认为是为了解决微前端全栈中不能在多个子应用间共享资源的问题，但联邦模块出现后又并不仅仅局限于微前端。
模块联邦允许多个`webpack`构建一起工作。从运行时的角度来看，多个构建的模块将表现得像一个巨大的连接模块图。从开发者的角度来看，模块可以从指定的远程构建中导入，并以最小的限制来使用。使`JavaScript`应用得以在客户端或服务器上动态运行或者动态加载另一个`bundle.js`或者`build`之后生成的代码，且共享依赖。

通过 `Module Federation`实现的代码共享是双向的，但每种情况都有降级方案。`Module federated`可以总是加载自己的依赖，但在下载前会去尝试使用消费者的依赖。更少的代码冗余，依赖共享就像一个单一的`Webpack`构建。

## 二.如何用联邦模块实现跨应用的代码共享

### <u>动态导入远程模块</u>

> 当远程项目不能固定写在webpack配置中时，可以通过动态导入的方式使用联邦模块

动态导入和静态导入的区别在于，需要自己实现js加载和容器连接

```ts
// host
// 加载js标签
function loadScript(src){
  return new Promise((res, rej) =>{
      const srcirpt = document.createElement('script')
      script.src = src;
      script.onload = res;
      script.onfaild = rej
      document.body.appendChild(script);
  })
}

// 连接容器
function loadComponent(scope, module) {
    return async () => {
      // 初始化共享作用域（shared scope）用提供的已知此构建和所有远程的模块填充它
      await __webpack_init_sharing__('default');
      const container = window[scope]; // 或从其他地方获取容器
      // 初始化容器 它可能提供共享模块
      await container.init(__webpack_share_scopes__.default);
      const factory = await window[scope].get(module);
      const Module = factory();
      return Module;
    };
}

// 加载远程模块的入口文件并并连接
loadScript('http://localhost:3001/remote-entry.js').then(() => {
  loadComponent('app', 'button').then(module => {
      console.log(module)
  });    
})

// remote
module.exports = {
  plugins: [new ModuleFederationPlugin({
      name: "app",
      filename: 'remote-entry.js',
      exposes: {
          "button": "./src/button.js"
      },
  })]
}
```

[动态导入远程模块demo地址](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FBxiaoyao%2Fdynamic-remotes)

### <u>静态导入远程模块</u>

```ts
// host
module.exports = {
  plugins: [new ModuleFederationPlugin({
      name: "host",
      remotes: {
          "app": "app@http://localhost:3001/remote-entry.js"
      },
      shared: {
          react: {
              eager: true
          }
      }
  })]
}

// remote
module.exports = {
  plugins: [new ModuleFederationPlugin({
      name: "app",
      filename: 'remote-entry.js',
      exposes: {
          "button": "./src/button.js"
      },
      shared: ['react']
  })]
}
```

[静态导入远程模块demo地址](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FBxiaoyao%2Fstatic-remotes)

### <u>联邦模块的三个主要功能</u>

- remotes 用来声明会引用哪些远程文件，以及从哪里来导入远程资源

```ts
// 在此项配置中，会从远程的 http://localhost:3000/app-entry.js 读取文件来加载 app 模块
new ModuleFederationPlugin({
    name: "template",
    remotes: {
        app: "http://localhost:3000/app-entry.js"
    }
})
```

- exposes 用来声明会提供哪些资源供远程使用

```ts
// 此项配置会相当于建立一个新的entry入口从 ./src/button.js 文件开始进行打包生产新的资源
// Tips: 
//   从这个entry打包的资源不会与webpack主入口的资源共享内容。即两个入口都引用了react的话，react会在两个项目中分别存在
new ModuleFederationPlugin({
    name: "template",
    exposes: {
        button: "./src/button.js"
    }
})
```

- shared

```ts
// 什么哪些模块会在主项目和远程项目中共享
// - 例如：当主项目提供了react的话，远程项目则会直接使用主项目的react模块，而不会再次从远程加载（远程项目的react会单独打包）
// - 主项目的shared集合需要包含远程项目的shared
// - eager: 是否立即加载模块而不是异步加载。如果在主项目的入口文件中依赖了这个模块就必须设置eager，否则报错
// - singleton: 是否确保使用单例模式
new ModuleFederationPlugin({
    name: "template",
    shared: {
        react: {
            eager: true
        }
    }
})
```

如果想看更多关于联邦模块的案例，可以访问[官方仓库](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fmodule-federation%2Fmodule-federation-examples)。

## 三.联邦模块的原理分析

联邦模块是通过webpack原生提供的`ModuleFederationPlugin`插件来实现的，它有两个主要概念：Host（消费其他 Remote）和Remote（被 Host 消费）。每个项目可以是Host也可以是Remote，也可以两个都是。

- 作为Host需要配置remote列表和shared模块。
- 作为Remote需要配置项目名（name），打包方式（library），打包后的文件名（filename），提供的模块（exposes），和Host共享的模块（shared）。

### <u>webpack打包原理</u>

联邦模块是基于webpack做的优化，所以在深入联邦模块之前我们首先得知道webpack是怎么做的打包工作。

通过阅读webpack打包结果可以了解到，webpack每次打包都会将资源全部包裹在一个立即执行函数里面，这样虽然避免了全局环境的污染，但也使得外部不能访问内部模块。

在这个立即执行函数里面，webpack使用 `__webpack_modules__` 对象保存所有的模块代码，然后用内部定义的__webpack_require__ 方法从 `__webpack_modules__` 中加载模块。并且在异步加载和文件拆分两种情况下向全局暴露一个 webpackChunk 数组用于沟通多个webpack资源，这个数组通过被webpack重写push方法，会在其他资源向webpackChunk 数组中新增内容时同步添加到 `__webpack_modules__`  中从而实现模块整合。

联邦模块就是基于这个机制，修改了 `__webpack_require__`  的部分实现，在require的时候从远程加载资源，然后合并到`__webpack_modules__` 中。

**Tips：**

- webpackChunk 可以通过 output.chunkLoadingGlobal 配置修改

### webpack构建资源

JS的模块化机制有许多的标准，不过在webpack中全部统一转化为自己实现的`__webpack_require__`

### webpack单文件结构

在webpack单文件结构中，所有模块的源码都存储在 `__webpack_modules__` 对象中，然后都统一使用内部定义的 `__webpack_require__` 方法来加载模块。

在这种情况下，外部完全无法访问内部的运行资源

```ts
(() => {
    var __webpack_modules__ = {
        './src/other.js': () => {
			// ...
        } 
    }
    
    var __webpack_module_cache__ = {} // 模块导出结果的缓存
    
    function __webpack_require__() {  // 内部模块导入方法
    	// .....
    } 
    // .....
    
    
    // 入口文件内容
    (() => {
        __webpack_require__('./src/other.js')
    })()
})()
```

### webpack多文件结构

在多文件结构中，存在两个地方存储模块源码，一个是内部的 `__webpack_modules__` 对象，另一个是全局的 `webpackChunk` 数组。

webpackChunk 数组其实是起一个桥梁作用，用于将其他文件中的模块加载到入口文件的 `__webpack_modules__` 对象中，而这是通过重写 webpackChunk数组的push方法实现的。

```ts
// index.js
(() => {
    var __webpack_modules__ = {
        './src/other.js': () => {
			// 源码
        } 
    }
    
    var __webpack_module_cache__ = {} // 模块导出结果的缓存
    
    function __webpack_require__() {} // 内部模块导入方法
    
    // 向全局暴露出一个对象
    self["webpackChunk"].push = webpackJsonpCallback.bind(null, chunkLoadingGlobal.push.bind(chunkLoadingGlobal));
    // 入口文件内容
    (() => {
        __webpack_require__('./src/other.js')
    })()
})()
```

```ts
// ./src/other2.js
(self["webpackChunk"] = self["webpackChunk"] || []).push([
  ["src_other2_js"],
  {
    "./src/other2.js": () => {
      console.log("in other2.js");
    }
  }
]);
```

### <u>ModuleFederationPlugin 的用法</u>

```ts
new ModuleFederationPlugin({
  name: "app1",
  library: { type: "var", name: "app1" },
  filename: "remoteEntry.js",
  remotes: {
    app2: 'app2',
    app3: 'app3',
  },
  remoteType: 'var',
  exposes: {
    antd: './src/antd',
    button: './src/button',
  },
  shared: ['react', 'react-dom'],
  shareScope: 'default'
})
```

配置属性：

- name: 必须，唯一 ID，作为输出的模块名（容器），使用的时通过 name/{name}/name/{expose} 的方式使用；
- library: 可选，打包方式，默认{ type: "var", name: options.name }，其中这里的 name 为作为 umd 的 name，是挂载在全局下的变量名；
- filename: 可选，打包后的文件名；
- remotes: 可选，表示当前应用是一个 Host，可以引用 Remote 中 expose 的模块；
- remoteType: 可选，默认var，("var"|"module"| "assign"|"this"|"window"|"self"|"global"|"commonjs"|"commonjs2"| "commonjs-module"|"amd"|"amd-require"|"umd"|"umd2"|"jsonp"|"system"|"promise"|"import"|"script")，远程容器的外部类型；
- exposes:可选，表示当前应用是一个 Remote，exposes 内的模块可以被其他的 Host 引用，引用方式为 import(name/{name}/name/{expose})；
- shared: 可选，主要是用来避免项目出现多个公共依赖，若是配置了这个属性，webpack在加载的时候会先判断本地应用是否存在对应的包，若是不存在，则加载远程应用的依赖包；
- shareScope，可选，用于所有共享模块的共享作用域名称

```ts
// 公共依赖shared的配置项
Shared = string[] | {
  [string]: {
    eager?: boolean; // 是否立即加载模块而不是异步加载
    import?: false | SharedItem; // 应该提供给共享作用域的模块。如果在共享范围中没有发现共享模块或版本无效，还充当回退模块。默认为属性名
    packageName?: string; // 设置包名称以查找所需的版本。只有当包名不能根据请求自动确定时，才需要这样做（如要禁用自动推断，请将requiredVersion设置为false）。
    requiredVersion?: false | string; // 共享范围内模块的版本要求
    shareKey?: string; // 用这个名称在共享范围中查找模块
    shareScope?: string; // 共享范围名称
    singleton?: boolean; // 是否在共享作用域中只允许共享模块的一个版本 (单例模式).
    strictVersion?: boolean; // 如果版本无效则不接受共享模块(默认为true，如果本地回退模块可用且共享模块不是一个单例，否则为false，如果没有指定所需的版本则无效)
    version?: false | string; // 所提供模块的版本，将替换较低的匹配版本
  }[]
}
```

**Tips：**

- 在使用 Module Federation 的时候一定要记得，将公共依赖配置到 shared 中。另外，一定要两个项目同时配置 shared ，否则会报错
- 入口文件index.js本身应该没有什么逻辑，将逻辑放在bootstrap.js中，index.js去动态加载bootstrap.js。如果将逻辑直接放到index.js里，会报错，如果是公共依赖配置shared报错，可以配eager参数来解决。主要原因是如果直接在index.js执行逻辑，会依赖Remote暴露的js，此时remote.js还没有加载，就会有问题。

### <u>ModuleFederationPlugin 的原理</u>

`ModuleFederationPlugin`主要做了三件事：

- 如何共享依赖：使用 SharePlugin
- 如何公开模块：使用 ContainerPlugin
- 如何引用模块：使用 ContainerReferencePlugin

### SharePlugin

该插件使公共依赖可共享

### ContainerPlugin

该插件为指定的公开模块创建entry。entry.js执行后会在window上挂一下对象，该对象有两个方法，get和init。get方法用来获取模块。init方法用来初始化容器，它可以提供共享模块。

```ts
// entry.js
var remote;remote = (() => {
    // ...
})()
```

```ts
// remote对象里的get和init方法
var get = (module, getScope) => {
	__webpack_require__.R = getScope;
	getScope = (
		__webpack_require__.o(moduleMap, module)
			? moduleMap[module]()
			: Promise.resolve().then(() => {
				throw new Error('Module "' + module + '" does not exist in container.');
			})
	);
	__webpack_require__.R = undefined;
	return getScope;
};
var init = (shareScope, initScope) => {
	if (!__webpack_require__.S) return;
	var oldScope = __webpack_require__.S["default"];
	var name = "default"
	if(oldScope && oldScope !== shareScope) throw new Error("Container initialization failed as it has already been initialized with a different share scope");
	__webpack_require__.S[name] = shareScope;
	return __webpack_require__.I(name, initScope);
};

// This exports getters to disallow modifications
__webpack_require__.d(exports, {
	get: () => get,
	init: () => init
});
```


在使用Remote的模块时候，通过init将自身shared写入Remote中，再通过get获取Remote中expose的组件，而作为Remote时，判断Host中是否有可用的共享依赖，若有，则加载Host的这部分依赖，若无，则加载自身依赖。

### ContainerReferencePlugin

该插件将特定的引用添加到作为外部资源（externals）的容器中，并允许从这些容器中导入远程模块。在导入时会调用容器使用者提供的remote进行重载。

> 通过remotes定义的模块，也会在 __webpack_modules__ 中声明但不会有具体实现，这就和异步导入类似。
> webpack5中新增了__webpack_require__.e 方法，对通过次方法导入的模块执行一下三个函数，并且全部成功才返回。
> - __webpack_require__.f.consumes 用来判断和消费shared模块，如果当前环境下已经有这个模块就不向远程请求
> - __webpack_require__.f.remotes 用来连接容器
> - __webpack_require__.f.j 用来加载JS


四.使用场景

1. 适用于新建专门的组件应用服务来管理全部组件和应用，其余业务层只须要根据本身业务所需载入对应的组件和功能模块便可

2. 模块管理统一管理，代码质量高，搭建速度快。适用于矩阵app，或者可视化页面搭建等场景

参考文章

[webpack 5 官方文档](https://link.juejin.cn/?target=https%3A%2F%2Fwebpack.docschina.org%2Fconcepts%2Fmodule-federation%2F)

[Webpack5 跨应用代码共享 - Module Federation](https://link.juejin.cn/?target=https%3A%2F%2Fsegmentfault.com%2Fa%2F1190000024449390)

[尝试webpack5 Module Federation](https://link.juejin.cn/?target=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F141390589)

[探索 webpack5 新特性 Module federation 引发的javascript共享模块变革](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fyingyangxing%2Farticle%2Fdetails%2F109653116)

[三大应用场景调研，Webpack 新功能 Module Federation 深入解析](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.aliyun.com%2Farticle%2F755252)