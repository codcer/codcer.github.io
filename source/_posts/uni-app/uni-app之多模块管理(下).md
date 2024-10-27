---
title: uni-app之多模块管理(下)
tags: [uni-app,跨端开发]
slug: 2f9lyb5
keywords: uni-app,多端同构,多端开发技巧,跨端开发
date: 2024-10-26 17:18:41
---

# 前提

有时候，app进行了大规模的调整，导致之前所有版本的app都不可用，或者一些重要功能作出了调整（比如收费内容发生改变），强制用户需要更新app，这样的情况并不少见；

因此在第一版本的app内，就应该把包内更新的功能加上，以保证app的后续进行增量。【早期就应该规划】

## app更新分类

整包更新和热更新。

1. 整包更新是指下载完整apk文件进行覆盖安装。

2. 热更新是指把app有改动的地方打包进wgt文件，只更新wgt文件中的内容，不进行整包安装，在用户视角也叫做省流量更新

## uni-app下app热更新方案

uni的app业务越来越庞大,同时协同的业务组越来越多. app的动态化就必须提上日程了.
对于app热更新,首当其冲的问题就是差异包.

因为我们依赖了很多的三方库.这些代码就必须在分包的时候单独剥离出来.
业务包让他纯粹的只有业务代码. 这样就可以保证业务包的体积比较小,**保证热更新时候的速度**.

注：需要注意的是，ios并不存在下载安装包覆盖安装这种操作，所以在ios平台需要跳转到appStore进行更新.

```js
    getVersion({
      platform: '1'
    }).then(res => {
      if (!res) {
        return;
      }
      
      // 版本号x.y.z
      const appCode = parseInt(res.app_code.split('.').join(''))
      const version = parseInt(widgetInfo.version.split('.').join(''))
  
      if (appCode > version) { 
        if (res.type === 0) { 
          updateHot(res.download_url)
        } else if (res.type === 1) { 
          updatePackage(res.download_url)
        }
      }
    }).catch(err => {
      console.error(err)
    });
```

### 主工程分包
之前我们有提到过我们有一个项目是主工程,里面基本没有任何的业务代码.只有一些代码运行需要的所有依赖.

因为我们的主工程与业务项目的依赖版本都是高度统一的.
所以我们node_modules下面的依赖包路径都是完全一致的.

uni-app页面文件的打包是基于`pages.json`，所以我们需要根据需求，使用node动态生成对应的`pages.json`文件。


工程分包文件示例:

```js
  // 类似这样的伪流程
  pagesJson = {
    pages: [...xx1Pages, ...xx2Pages, ...],
    subPackages: [...subPackagesXxx1, ...subPackagesXxx2, ...subPackagesXxx3],
    ...otherConfig
  };
 
  // 将最终的结果写入pages.json，在项目运行或构建时
  fs.writeFileSync(pagesPath, JSON.stringify(pagesJson));

```

### 业务模块的分包间依赖

因uni-app下的小程序，不支持分包中异步导入模块，所以这里在每个pages的页面下配置这段

```json
  // 略
  "usingComponents": {
    "widget-list": "@xxx/业务模块"
  },
  "componentPlaceholder": {
    "widget-list": "view"
  },
  // 略
```

```ts
  // 略
  const componentPlaceholder = {
    'widget-list': 'view',
  };
  // 略
```


## 分包部署与下发客户端

所有类型的包打完之后,我们会压缩成wgt包,部署到cdn上.
当客户端检测到有模块已经更新,则会从cdn上拉取相对应的wgt包.

## app加载顺序
app必须加载主工程的代码完成之后,才可以加载业务包.
这是必要条件,不然会直接闪退.

## 客户端热更新容错

为了保证客户端在任何时候都可以正常运行,我们会在客户端发版的时候将所有分包打进客户端中.当客户端热更新失败的时候,将会回滚到最初打进客户端的代码.

以下是可能会导致更新失败的场景,我们都会回滚渲染:

* 获取代码包超时

* 当md5不匹配时

* 当代码解压失败时

* 代码包版本号不合法

* 首次进入就闪退

还有很多场景,这里就不一一列出了.
但是这一块要绝对重视,一旦出错,就会导致客户端直接闪退.

## 尾巴
为了保证独立开发,独立部署,我们的CI/CD根据这一套打包方案也做了很多的定制与开发.这里就不展开篇幅来讲了,分包其实很简单,最重要的还是客户端的稳定性.

在用户手机上会出现非常多意想不到的极端环境.如何保证运行时的稳定性.

这里需要大家好好思考.