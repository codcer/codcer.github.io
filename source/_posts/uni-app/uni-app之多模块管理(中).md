---
title: uni-app之多模块管理(中)
tags: [uni-app,跨端开发]
slug: j4ksvfa8
keywords: uni-app,多端同构,多端开发技巧,跨端开发
date: 2024-10-26 17:18:39
---


## 实现一个简单的模块管理工具,解决npm拉取业务模块慢的问题

使用npm拉取git仓库,速度还是可以接受的.
主要慢的地方在于 `postinstall`的时候需要在npm拉取git仓库的时候,需要重新构建项目代码.还有一点就是业务模块也有依赖,npm install的时候也需要重新拉取相同的依赖.

这两个步骤其实是可以省略的,解决的办法就是可以实现一个简单的模块管理工具来替代npm加载业务模块的处理.


### 创建一个配置文件

```js
module.exports = {
    "Module1": {
        "url": "http://xxx1.git",
        "moduleName": [
            "模块名1"
        ],
        "branch": "dev",
        "type": "uni-app"
    },
    "Module2": {
        "url": "http://xxx2.git",
        "moduleName": [
            "模块名2"
        ],
        "branch": "dev",
        "type": "uni-app"
    }
}
```

因为每一个项目都会用到这一个文件,把这个文件放到服务器上.
在业务模块初始化的时候,自动拉取解析这个文件.再使用nodejs拉取相应的git仓库.

### 拉取git仓库

使用[`download-git-repo`](https://www.npmjs.com/package/download-git-repo)拉取git仓库的代码放到node_modules目录下.

这样在项目中,我就可以引用普通npm一样引入这个仓库的代码了.

### 模拟npm生命周期
代码拉下来之后,还需要用postinstall来构建业务代码,
所以需要使用nodejs遍历以下业务模块的目录,读取目录下的每一个packages.json.

如果有npm钩子的,就执行一下里面的script.

大致示例代码如下:

```js
    // 示例代码,仅供参考
    const spawn = require('cross-spawn');
    const fse = require('fs-extra');

    const packageData = fse.readJsonSync('./packages.json');

    if (packageData.scripts.preinstall) {
        spawn.sync('npm',['run','preinstall']);
    }

    if (packageData.scripts.postinstall) {
        spawn.sync('npm',['run','postinstall']);
    }

```

### 并发执行业务模块的npm钩子

在npm执行postinstall的时候,并不是全部一股脑的全部执行.

使用 yarn 安装依赖的时候,同时最多可以4个模块执行(npm install 没有具体研究).
所以构建业务模块的时候,总是会出现一个一个等的情况.

如果使用自己的模块管理工具,可以解除这一限制(当然,具体可以自己去go下).
要是业务模块实在太多,自己电脑撑不住的话,可以适当调整并发执行npm钩子的数量.

### 使用配置文件自动生成pages.json

上面我们提到,我们有一个管理业务模块的配置文件.
有了这个东西,我们就可以根据配置文件的内容动态生成pages.json的内容了.

## 尾巴

本篇介绍如何解除npm的限制,快速初始化项目的业务模块.
