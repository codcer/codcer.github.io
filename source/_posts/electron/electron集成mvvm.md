---
title: electron集成mvvm
tags: [electron]
slug: ea793fdc
keywords: electron
date: 2024-11-13 23:07:34
---

# 快速入门Electron  

## 与vue集成

```shell
    # vue-cli & ts | js
    vue create electron-demo 
    vue add electron-builder
    # 已内置vue-tool

    # vite git template
    git clone https://github.com/cawa-93/vite-electron-builder.git
    
    # vite-cli   
```

## 与react集成

```shell
    # 1: github上比较稳定的仓库webpack+react+electron模版初始化
    git clone --depth 1 --branch main https://github.com/electron-react-boilerplate/electron-react-boilerplate.git your-project-name
    
    cd your-project-name

    # 查看当前项目下包需要更新, 不过升级须谨慎
    ncu 
    
    # 2：vite-react-electron-template
    git clone https://github.com/electron-vite/electron-vite-react.git

```

## 国际化i18n

```shell

# 用法和web端的使用方式一样
pnpm install i18n

```
