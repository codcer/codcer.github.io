---
title: 虚拟任务栈
tags: [vue,源码]
slug: ea549f7c
keywords: vue,虚拟任务栈,源码
date: 2023-09-01 22:17:36
---

## 什么是虚拟任务栈

    1. 具备的进入页面动画、退出页面动画、进入页面读取数据渲染视图、返回之前页面无需重新读取数据渲染视图等特性
    2. 特点:
       1. 虚拟任务栈可以模拟 原生应用中 Task 栈的效果;
       2. 进出场页面动画;
       3. 进入页面读取数据渲染视图;
       4. 返回之前的页面 无需重新拉取数据渲染视图,而是缓存

## 该怎么弄, 才可达到以上效果

    1. 我们需要监听到路由的跳转（页面的跳转）
    2. 我们需要保存已经进入栈中的页面，而不是销毁他们

首先：需要监听到路由的跳转（页面的跳转）
    - 监听路由的跳转，VueRouter 为我们提供了现成的解决方案，我们可以直接通过 watch 属性来监听 $router 的跳转变化：

    ```js
        // 代码来自 VueRouter 官网：https://router.vuejs.org/zh/guide/advanced/transitions.html
        // watch $route 决定使用哪种过渡
        watch: {
            '$route' (to, from) {
                const toDepth = to.path.split('/').length
                const fromDepth = from.path.split('/').length
                this.transitionName = toDepth < fromDepth ? 'slide-right' : 'slide-left'
            }
        }
    ```

其次：需要保存已经进入栈中的页面，而不是销毁他们