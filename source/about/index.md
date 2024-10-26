---
title: 个人简历
abbrlink: f0b34752
date: 2024-10-26 14:31:51
---

# 联系方式

- Email：codcer@gmail.com

---

# 个人信息

 - liu/男/1989
 - 院校：南昌理工 2008.09-2012.07 
 - 专业：计算机科学与技术
 - 技术博客：https://codcer.github.io
 - Github：https://github.com/codcer
 - 期望职位：前端资深开发
 - 期望城市：上海

---

# 工作经历

1、立达信（2020.5 ~ 2024.8.31）

负责集团的前端架构、uni-app、前端基建、低代码引擎、共享物料层及业务开发, 疑难问题及性能、内存分析。

技术栈：react(vue)全家桶 + rxjs + ts+ webpack5 (vite) + uni-app + qiankun + mf + koa

主要负责：
  1. 解耦集团数字化系统，采用微前端架构策略对其进行重构和层次划分。在新的架构中，主应用负责提供基础能力，而子应用则独立运行，最终以组合方式集成至主应用。此过程中，成功集成了200多个React和Vue的内部及外部应用，实现了样式的隔离、子应用路由的缓存以及子应用多实例的缓存机制。
  2. 封装MQTT业务组件, 统一化多端,pc端,通过中间层自动生成订阅、回调,生命周期hoc,应用层只负责订阅的主题,数据由redux、vuex注入至组件层,由之前近百行代码,缩减至10行代码以及小程序分包自动化(分包策略,分包之间引入对方的库打包策略)。
  3. 低代码引擎, 通过表单配置方案(formily+designable), 按照物料的开发规范, 进行组件及物料的开发和按需打包。
  4. 多端采用低代码引擎+ uni-app(有限制)方式适配h5和app端,小程序上的低代码采用h5方式集成, 后台通过表单引擎配置; 小程序超包问题则采用分包和分包异步化(解决业务端引入不到分包中组件)。
  5. 共享物料层, 根据低代码引擎实现的生态, 加快打包速度,线上动态化载入物料,迎合微前端让业务端更加独立和控制发布粒度;结合低码使组件的复用更高效(借助import()和federation策略实现版本的控制和发版)。
  6. 参与过的业务开发主要包括基于konva开发的2D空间配置器和物联设备控制；高德地图轨迹和标点、空间下占；小程序上物联控制、用户选择器、信息发布和相册及视频发布等；
  7. 沉淀基建coral包, 解偶三方ui库升级导致迁移及统一的问题;
  bui基于antd封装的基础库, 业务的biz库、多端tui库、工具库, 并维护至私服包上，文档站点基于koa部署。

> `神州优车` 2018.5 - 2020.5

前端资深工程师 厦门

负责买买车业务系统的前端设计及改进，并参与后管系统、车主还款 H5 端、hybrid 开发；

负责系统优化，如打包，缓存，垫片等策略方面；

技术栈：vue + vuex + elementUI + cube-ui + webpack + 垫片（主要用于 H5 端）

主要负责：

- 车贷后管系统（vue + vuex + elementUI + webpack）

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

> `杭州铭师堂教育有限公司` 2016.5 - 2018.5

高级前端工程师 杭州

负责 OA 业务系统的前端设计及优化，OA 业务系统 H5、hybrid 端开发；

负责系统优化，如打包，缓存，垫片等策略方面；

参与其它业务线专题、h5 开发；

技术栈：react + redux + antd（PC）｜ antm（h5）、vue + vuex + vux + babel-polyfill 垫片（H5、hybrid 端）

主要负责：

- 后管系统（react + redux + antd + webpack + i18n）- PC 端 OA

  1. 组件、页面取消请求 高阶组件：通过钩子 constructor、componentWillUnmount、render 个钩子，在 constructor 生成 cancel，在 componentWillUnmount 执行 cancel, 在 render 返回高阶组件的属性代理；
  2. 无限级菜单生成 + 权限数据绑定：利用 api 返回菜单和权限数据， 菜单 Menu、SubMenu、Item 组件、路由 prop 传递权限拼装；页面级 provider、组件级 consumer；
  3. 面包屑 tab 导航生成，利用 redux redux-thunk 和 router，生成已打开菜单组，删除，右键批量操作更新相应的 redux 中 action -> reducer -> state。

- OA 业务系统 H5、hybrid 端开发 （vue + vuex + vux + babel-polyfill 垫片+pxtorem）- 移动端 OA

  1. 大数据量优化 - 虚拟列表优化；
  2. 员工地图定位、签到拍照打卡，基于高德地图 api 定位，定位准确率 97%；签到工作流缓存策略；
  3. 拍照上传压缩优化进度展示；压缩之后比例 500kb 之内；

- 其它业务线专题页、h5 开发 （react + redux + antd + webpack + MPA）
  1. 专题页 采取 MPA 方式即多入口页面开发，router 自动生成，依据 require.context 扫描目录生成 routes 表，主要是因为各个页面不相互关联，故采取这种 2B 方式开发；
  2. 页面 store 采取动态注入方式，主要通过的 getComponent, combineReducer Api；
  3. 做过的功能，如做题，题目解析，分析报告图表 echart；活动下单存储策略；多个块滚动吸附吸顶；

> `杭州粉团网络公司` 2012.4 - 2016.5

前端工程师 杭州

负责借贷后台系统的前端设计及优化，借贷 H5、hybrid 端开发，系统优化；

技术栈：react + redux + antd（PC）、jquery + html5(H5、hybrid 端)

---
## 开源项目

 - [v-vuex](https://github.com/codcer/v-vuex)：vuex模块化
 - [hijacking-webpack-plugin](https://github.com/codcer/hijacking-webpack-plugin) : HTML模板文件注入js脚本

---
## 技能清单
以下均为我熟练使用的技能

- 前端框架：React/Vue/qiankun/Rxjs/formily
- 前端工具：Scss/Less/webpack/vite/cra/vue-cli/Node
- UI框架：Antd/Element/vux/cube-ui/uview-ui
- 性能优化：ServiceWorker/Cache-Policy/module-federation/hybrid
- 后台框架：Koa/Nginx

---
## 自我评价

10年WEB前端开发经验。3年团队管理经验.

5年时间里带领团队完成众多项目,期间踩过无数技术坑与人员管理的坑.

近两三年来一直致力于项目的高度工程化,自动化的探索与实践.

也在十几名前端开发人员同时开发的大型项目中,完成了`前端微服务化`的构想与落地.

进而摆脱了当前前端技术栈的限制. 解决了大项目中多条产品线并行开发,部署的众多问题.

在立达信集团期间,带领团队完成了众多高难度项目,解决了众多技术与开发效率的问题.

也参与了公司的联邦共享库的开发.

希望可以遇到更有挑战性的工作,经历更多有趣的事情.

---

# 致谢
感谢您花时间阅读
期待能有机会与您共事。