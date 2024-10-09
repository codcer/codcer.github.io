---
title: 调试工具 - whistle
slug: t0uky1fwde
date: 2018-09-15 17:15:00
keywords: Gitlab,调试工具
tags: [debug]
---
[常规使用]
一、安装
  ```ts
  npm i -g whistle
  ```

二、启动或停止或重启
```ts
[i] whistle@2.7.12 started
[i] 1. use your device to visit the following URL list, gets the IP of the URL you can access:
       http://127.0.0.1:8899/
       http://192.168.1.8:8899/
       Note: If all the above URLs are unable to access, check the firewall settings
             For help see https://github.com/avwo/whistle
[i] 2. configure your device to use whistle as its HTTP and HTTPS proxy on IP:8899
[i] 3. use Chrome to visit http://local.whistlejs.com/ to get started
```

三、代理(手机或PC)、证书

[代理设置](proxy.png)
安装https 根证书信任

[高阶使用]
1. 请求转发(替换为本地文件)[不用等后端接口返回,可以模拟接口数据]
rules: http://api/obj/getList  file://...本地文件各种格式...
2. 请求转发[替换本地文件,代理线上至本地]
   `线上环境,有特殊问题,没法在生产复现,此时可直接替换js至本地进行调试`
3. 请求转发(替换为另一个线上路径)
   `跨域的一种解决方案`
4. 注入html、js、css
   `whistle会自动根据响应内容的类型,判断是否注入相应的文件及如何注入(是否要用标签包裹起来)` 

rules:
   http://www.baidu.com  html:///users/xxx/xx.html
   http://www.baidu.com  js:///users/xxx/xx.js
   http://www.baidu.com  css:///users/xxx/xx.css
values:
    create => 资源&实例化
    https://www|m.baidu.com  jsPrepend://{vconsole.min.js} # 引入源码

    https://www|m.baidu.com  jsPrepend://{vconsole.js} # 引入业务js
5. 使用log功能打印日志
    `移动端真机调试中,无法像浏览器console那样查看日志和数据,早前方案是注入vconsole插件,而有了whistle后,我们可以这样做, 当我们访问手机端页面log,就回输出到whistle中`

    www|m.baidu.com log:// #whistle中log记录

6. 跨域,这样后端不用在设置头
   `https:后端接口地址  resCors://*`
