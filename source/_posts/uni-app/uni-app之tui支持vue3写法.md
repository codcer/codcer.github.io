---
title: uni-app之tui支持vue3写法
tags: [uni-app,跨端开发]
slug: wwu086as
keywords: uni-app,多端同构,多端开发技巧,跨端开发
date: 2024-10-26 17:20:36
---


## uni-app UI 不支持vue3的窘境

uni-app UI 社区上一直没有比较好的兼容vue3语法的ui库，经过调研后，发现有个个人版的`vk-uview-ui`这个库，
因此基于此，我们下沉的tui来适配vue3风格【自己动手丰衣足食 🤒】。

### 组件库打包与发布

如果组件库还是按照原来的打包与发布模式下.在uni-app中下，不能完全行，受制于uni-cli在微信处理的时候，走的是静态
编译，导致了我们tui库采用自定义打包和发布模式。 

组件库入口下仅对全局的api和方法，进行初始化。
另外所有的组件都放在了components目录下，不对这些组件暴露至入口js下，这样做可以解决，vite模式下，首屏大量的请求
的问题。


最后，经过我们自定义处理后，就可以愉快的发布npm包至私服上。


## 如何使用这样的组件库

在业务开发的时候,代码只要直接引入这个npm包就好了；另外在uni的easy-com方式和tsconfig.json下配置这个库的类型生命，
按需引入即可和类型提示就也有。




