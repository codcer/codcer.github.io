---
title: uni-app之依赖管理
tags: [uni-app,跨端开发]
slug: h8gas5c
keywords: uni-app,多端同构,多端开发技巧,跨端开发
date: 2024-10-26 17:17:36
---

目前我们通过uni-app触达的端已经有:

* 微信小程序
* app
* H5

为了快速的让业务可以触达到各端,我们每一个业务模块都是独立的.

他们根据业务需要,分别兼容到不同的端,等到构建的时候再将其进行组装成为一个完整的项目.

为了满足以上需要,并且可以灵活开发.所以每一个业务模块都是可以独立开发,独立运行的,可以不依赖主工程.

目前来说项目之间的关系已经比较类似`微前端`了.

因为大部分端都不能做到热更新,所以相对微前端而言小程序们做不到`独立部署`.

> 关于uni-app -> app部分,后面会有专门的文章跟大家介绍我们是怎么玩的.

今天就跟大家介绍一下,多业务,多团队,多端的代码管理中我们遇到的第一个问题: `依赖管理问题`


## 所有项目依赖统一管理

因为业务模块很多,我们项目工程的差异必须保持高度统一,其中包括依赖.

### 核心依赖集中管理

现在我们团队开发的跨端模块已经有几十个.为了方便管理,

我将所有的依赖封装成了一个`npm包`,里面没有任何的业务代码,只有稳定的依赖.

所有的项目都会引入这个核心的依赖,以保证所有项目可以稳定运行.

### 非核心依赖版本管理问题

如果是非核心依赖,我们将使用我们自研的`cli`工具强行修改package.json里面的版本号.

持续集成的过程中,所有的跨端项目都会通过这个cli工具在`npm install`执行之前修改我们的项目配置.其中就包括了我们的依赖.

这样所有的跨端项目都会更新到最新的依赖配置.

### 个别项目特殊处理办法

所有项目的依赖全部统一之后,总会有一些特殊原因个别项目的依赖会有一点区别.

这里就必须要提到`package.json`的 `resolutions`属性.

`resolutions` 字段用于解析选择性版本，可以通过此功能自定义依赖版本。 

这样npm就会将多版本共存的版本,强行指定某一版本,满足个别项目的特定需求.

如果我要所有的 `PB` 强行指定2.0

使用方法如下:

```json
// package.json 
// 这样PB这个依赖库,就被强行指定版本了
{
    "resolutions": {
        "PB": "2.0.0"
    }
}
```

resolutions的解释,你可以在这里查看更多:

https://classic.yarnpkg.com/zh-Hans/docs/selective-version-resolutions

当然你也可以使用这个工具,将依赖强制指定版本:

https://www.npmjs.com/package/npm-force-resolutions



## 第三方依赖稳定性问题

我们在开发周期比较长的前端项目的时候,必然会遇到依赖管理的问题.

我们在开发项目的时候,我们用了大量的三方库.这些三方的依赖库时不时的会更新自己的代码.

第三方依赖库的代码更新会很容易造成代码运行的不稳定,

比如昨天还跑的好好的项目,另一位刚刚接手的同学重新安装依赖之后项目就完全跑不起来了.

或者自己机器跑的好好的代码,扔到打包机上重新打包之后就完全跑不起来.

因为三方依赖库用的太多,很多时候并不能很快速的定位到项目为什么跑不起来.

一个依赖更新的问题可能会定位一个大半天才能找到罪魁祸首(曾经的我为了这种事情浪费了大量的青春).


## 锁定NPM依赖版本

对于那些你不熟悉,或者不能保证稳定的三方npm包,我这边的做法是`直接锁定版本号`.

这件事情做的越早越好,不要盲目相信三方npm越更新性能越好.

他会时不时的让你痛一下.让你浪费大量的时间去寻找到底是谁影响了你的代码正常运行.


## 第三方npm包有bug如何解决?

### 升级原有npm包版本
如果你发现了你的第三方npm有bug,建议不要盲目的升级版本.

第一步你需要先看这个npm包的 `changelog`,如果有明确修复过这个bug,

才会建议你升级.但是不能保证升级之后可能会带来其他bug.

### 修掉bug发布在公司私有源

但是我还是比较建议,在原来版本的npm包上修掉这个bug,并且修改npm包名为公司内部包,发布到公司私有npm源上. 

如果因为npm版本的改动,导致项目不稳定,也可以缩小排查范围,快速定位到是哪一个npm库导致的问题.

例如: mobx => @公司源/mobx


### 小技巧: .npmrc配置

如果你有发布到公司私有源的三方npm包,

你可以尝试修改.npmrc文件使用npm名的作用域直接指向公司私有源

例如:

```bash
@公司源:registry=http://npm.xxx.com
@coral-appjs:registry=http://npm.xxx.com
```

## 版本变更后的开发环境和生产环境

如果你目前版本变更的比较频繁的话,

很容易产生版本变更后两个环境的不一致,很有可能会产生一些微妙的问题.

在这里我还是建议: 每次升级后还是完全的删除 `node_modules` 目录,重新安装依赖.特别是在打包机上.

安装依赖的实践,相对你排查莫名其妙的问题的实践,其实是`赚的`. 

当然,这只是在版本变更比较频繁的情况下,如果开始组件稳定,

打包机上还是建议开始缓存依赖,这样可以减少大量日常开发时间.


## 关于依赖安装加速

除了大家都知道的npm换源之外,npm依赖的执行文件下载也会拖慢你的依赖安装速度.

以下是我们团队使用的`.npmrc`配置,对于一些npm依赖的执行文件,
可以使用该配置加速你的npm构建速度.

你也可以安装 `mirror-config-china` 来加速你的依赖安装速度

https://www.npmjs.com/package/mirror-config-china 

> .npmrc文件的位置在你的home目录下,你也可以在项目根目录创建该文件.

这是我们常用的`.npmrc`配置,我们已经将它集成到我们的cli工具中,每当cli执行都会更新其配置.

保证每一个小伙伴的开发机以及打包机都是一致的.

```
chromedriver-cdnurl=https://npm.taobao.org/mirrors/chromedriver
couchbase-binary-host-mirror=https://npm.taobao.org/mirrors/couchbase/v{version}
debug-binary-host-mirror=https://npm.taobao.org/mirrors/node-inspector
electron-mirror=https://npm.taobao.org/mirrors/electron/
flow-bin-binary-host-mirror=https://npm.taobao.org/mirrors/flow/v
fse-binary-host-mirror=https://npm.taobao.org/mirrors/fsevents
fuse-bindings-binary-host-mirror=https://npm.taobao.org/mirrors/fuse-bindings/v{version}
git4win-mirror=https://npm.taobao.org/mirrors/git-for-windows
gl-binary-host-mirror=https://npm.taobao.org/mirrors/gl/v{version}
grpc-node-binary-host-mirror=https://npm.taobao.org/mirrors
hackrf-binary-host-mirror=https://npm.taobao.org/mirrors/hackrf/v{version}
leveldown-binary-host-mirror=https://npm.taobao.org/mirrors/leveldown/v{version}
leveldown-hyper-binary-host-mirror=https://npm.taobao.org/mirrors/leveldown-hyper/v{version}
mknod-binary-host-mirror=https://npm.taobao.org/mirrors/mknod/v{version}
node-sqlite3-binary-host-mirror=https://npm.taobao.org/mirrors
node-tk5-binary-host-mirror=https://npm.taobao.org/mirrors/node-tk5/v{version}
nodegit-binary-host-mirror=https://npm.taobao.org/mirrors/nodegit/v{version}/
operadriver-cdnurl=https://npm.taobao.org/mirrors/operadriver
phantomjs-cdnurl=https://npm.taobao.org/mirrors/phantomjs
profiler-binary-host-mirror=https://npm.taobao.org/mirrors/node-inspector/
puppeteer-download-host=https://npm.taobao.org/mirrors
python-mirror=https://npm.taobao.org/mirrors/python
rabin-binary-host-mirror=https://npm.taobao.org/mirrors/rabin/v{version}
sass-binary-site=https://npm.taobao.org/mirrors/node-sass
sodium-prebuilt-binary-host-mirror=https://npm.taobao.org/mirrors/sodium-prebuilt/v{version}
sqlite3-binary-site=https://npm.taobao.org/mirrors/sqlite3
utf-8-validate-binary-host-mirror=https://npm.taobao.org/mirrors/utf-8-validate/v{version}
utp-native-binary-host-mirror=https://npm.taobao.org/mirrors/utp-native/v{version}
zmq-prebuilt-binary-host-mirror=https://npm.taobao.org/mirrors/zmq-prebuilt/v{version}
sentrycli_cdnurl=https://npm.taobao.org/mirrors/sentry-cli
```

## 尾巴

今天给大家介绍了我们的依赖如何管理.

因为我们可以统一我们所有项目细节,因此在开发过程中减少了太多的不必要麻烦.

这也是我们方便管理项目最为重要的步骤之一.

如果大家对我们`cli`实现了哪些功能感兴趣,我可以单独出一篇聊一聊~