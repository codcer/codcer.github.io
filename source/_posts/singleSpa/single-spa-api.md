---
title: single-spa API 研读
tags: [微前端,MicroFrontend,前端全栈]
slug: ea799f7c
keywords: 微前端,前端微服务化,前端自动化,解决方案,前端难题,Single,SPA
date: 2018-09-01 22:17:36
---

Single-spa(简称sspa) 是一个将多个单页应用聚合为一个整体应用的js微前端框架;

> 好处:
    集成多个技术栈 而不刷新页面
    独立部署应用
    新功能使用新框架，旧的不重写可以共存
    改善初始加载时间，迟加载代码

## 架构

    1. sspa完整生命周期(bootstrap,mount,umount);
    2. sspa活动下会响应url路由,dom init -> mount -> unmount自身;
    3. 非活动下,DOM移除
    4. 核心加载文件

    ```js
    // main.js
    import * as singleSpa from 'single-spa';
    const name = 'app1';
    const app = () => import('./app1/app1.js');
    const activeWhen = '/app1';
    singleSpa.registerApplication({ name, app, activeWhen });
    singleSpa.start();

    // app1.js
    let domEl;
    export function bootstrap(props) {
        return Promise
            .resolve()
            .then(() => {
                domEl = document.createElement('div');
                domEl.id = 'app1';
                document.body.appendChild(domEl);
            });
    }
    export function mount(props) {
        return Promise
            .resolve()
            .then(() => {
                // 在这里通常使用框架将ui组件挂载到dom。请参阅https://single-spa.js.org/docs/ecosystem.html。
                domEl.textContent = 'App 1 is mounted!'
            });
    }
    export function unmount(props) {
        return Promise
            .resolve()
            .then(() => {
                // 在这里通常是通知框架把ui组件从dom中卸载。参见https://single-spa.js.org/docs/ecosystem.html
                domEl.textContent = '';
            })
    }
    ```

## sspa cli

    [官方脚手架cli](https://zh-hans.single-spa.js.org/docs/create-single-spa)
npm install --global create-single-spa
--dir|moduleType|framework

## 微前端类型

在single-spa中，有以下三种微前端类型：(一个应用可能包含多种类型)

 1. single-spa applications:有一组特定路由,渲染组件的微前端。
 2. single-spa parcels: 不受路由控制，渲染组件的微前端。
 3. utility modules: 非渲染组件，用于暴露共享js逻辑的微前端。
具体见:[官方解释](https://zh-hans.single-spa.js.org/docs/module-types)      
