---
title: About Me
date: 2020-06-25 14:31:51
---

# 技术方向: Front End Developer ---> Full Stack Developer

## 个人简历

性别：男 出生地：江西·上饶 生日：1989.01

电子邮箱：codcer@gmail.com | 2969206932@qq.com

毕业院校：南昌理工 2008.09-2012.07 专业：计算机科学与技术

所在地: 厦门

## 专业技能

- 熟悉 HTML5、CSS、JavaScript、TypeScript、Ajax、PWA、性能优化、模块化、node、自动化、gulp、i18n、前端规范（eslint+pritter+husky+changelog+git cz） 等相关 web 前端技术；
- 熟悉常用的 UI 框架（element-ui、antd、antm、cube-ui、vux）等其它 ui 框架的使用；
- 熟悉浏览器、H5 兼容性，bug 分析、定位，编写底层封装、webpack 插件来提升打包、性能及体验等；
- 熟练使用 webpack 搭建开发 H5(各种适配方案)、PC 端、Hybird、SPA、MPA 等自定环境；
- 熟练 MVC、发布-订阅等设计模式、主流技术栈全家桶完成开发；
- 熟练 MVVM 框架开发（VUE、React）及生态（vuex、saga、redux、redux-thunk）运用等；
- 发布 npm：v-vuex【vuex model 化】、postcss-pxtovw-includes【针对特定目录转 vw】、hijacking-webpack-plugin【防 DNS 劫持】;j 具体见本人 github（juhttp://github.com/codcer）;
- 对性能优化，前端全栈方案的持续探究等。（长任务-time-slice，请求过度-axios-cancelToken，微前端服务-simple-spa-可落地）。

## 项目经验

> `神州优车` 2019.7 - 至今

前端工程师 福建~厦门

负责买买车业务系统的前端设计及改进，并参与后管系统、车主还款 H5 端、hbiryd 开发；

负责系统优化，如打包，缓存，垫片等策略方面；

带新人；

负责前端埋点监控系统研发，技术基建开发。（个人发起）【开发中】

技术栈：vue + vuex + elementUI + cube-ui + webpack + 垫片（主要用于 H5 端）

主要负责：

- 后管系统（vue + vuex + elementUI + webpack）

  1. 底层封装：axios 拦截器配置，全局防重复请求，主要利用 axios.cancelToken 生成对应 cancel 方法，利用 hash（req.url + 参数哈希 + method）标记唯一性，队列中存在相同的请求，后入队列请求 cancel，请求结束移除对应的请求；
  2. 组件取消请求 mixin：通过钩子 created、beforeDestroy2 个钩子，在 created 生成 cancel，在 beforeDestroy 执行 cancel；
  3. 无限级菜单生成 + 权限数据绑定：利用 api 返回菜单和权限数据，和 elementUI 菜单组件、路由传递权限拼装；页面级 provide、组件级 inject；
  4. tab 导航生成，利用 v-vuex、vuex 全局和 router，生成已打开菜单组，删除，右键批量操作更新相应的 vuex 中 mutaion、action;
  5. 封装动态表单及动态校验；

- 车主还款 h5 端 + hybrid 端（ios、安卓）（vue + vuex + cube-ui + webpack + 垫片 + PWA + pxtovw）

  1. 封装一套代码适用 h5、hybrid 端，jsbridge 封装（js 桥、数据加密、解密封装）、动态惰性切换请求模式；
  2. 三方支付（支付宝、银联、云闪付、微信）app 和 h5 端对接；
  3. 为了解决产品要求一致性，重写键盘界面 + 相对复杂的交互；
  4. 垫片采取 core-js 按需导入方式解决浏览器 api 不兼容问题；
  5. PWA 策略：静态资源通过缓存优先，接口资源通过网络优先策略；online、offline 提醒；添加桌面应用；

> `杭州铭师堂教育有限公司` 2016.5 - 2019.7

前端工程师(业务负责人) 浙江~杭州

负责 OA 业务系统的前端设计及优化，OA 业务系统 H5、hbiryd 端开发；

负责系统优化，如打包，缓存，垫片等策略方面；

参与其它业务线专题、h5 开发；

技术栈：react + redux + antd（PC）｜ antm（h5）、vue + vuex + vux + babel-polyfill 垫片（H5、hybrid 端）

主要负责：

- 后管系统（react + redux + antd + webpack + i18n）- PC 端 OA

  1. 组件、页面取消请求 高阶组件：通过钩子 constructor、componentWillUnmount、render 个钩子，在 constructor 生成 cancel，在 componentWillUnmount 执行 cancel, 在 render 返回高阶组件的属性代理；
  2. 无限级菜单生成 + 权限数据绑定：利用 api 返回菜单和权限数据， 菜单 Menu、SubMenu、Item 组件、路由 prop 传递权限拼装；页面级 provider、组件级 consumer；
  3. 面包屑 tab 导航生成，利用 redux redux-thunk 和 router，生成已打开菜单组，删除，右键批量操作更新相应的 redux 中 action -> reducer -> state。

- OA 业务系统 H5、hbiryd 端开发 （vue + vuex + vux + babel-polyfill 垫片+pxtorem）- 移动端 OA

  1. 大数据量优化 - 虚拟列表优化；
  2. 员工地图定位、签到拍照打卡，基于高德地图 api 定位，定位准确率 97%；签到工作流缓存策略；
  3. 拍照上传压缩优化进度展示；压缩之后比例 500kb 之内；

- 其它业务线专题页、h5 开发 （react + redux + antd + webpack + MPA）
  1. 专题页 采取 MPA 方式即多入口页面开发，router 自动生成，依据 require.context 扫描目录生成 routes 表，主要是因为各个页面不相互关联，故采取这种 2B 方式开发；
  2. 页面 store 采取动态注入方式，主要通过的 getComponent, combineReducer Api；
  3. 做过的功能，如做题，题目解析，分析报告图表 echart；活动下单存储策略；多个块滚动吸附吸顶；

> `杭州粉团网络公司` 2012.4 - 2016.5

前端工程师(前端 leader) 浙江~杭州

负责借贷后台系统的前端设计及优化，借贷 H5、hbiryd 端开发，系统优化；

技术栈：react + redux + antd（PC）、jquery + html5(H5、hybrid 端)

## 友情链接
 
-