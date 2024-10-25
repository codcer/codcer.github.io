---
title: single-spa 流程
tags: [微前端,MicroFrontend,前端全栈]
slug: ea799f7c
keywords: 微前端,前端微服务化,前端自动化,解决方案,前端难题
date: 2018-09-01 22:17:36
---
## single-spa主流程解析

首先， 要明确两个点：

第一, single-spa是通过event-loop来控制每个应用之间的状态切换的（invoke函数）

第二, single-spa是就像一个状态机，通过给应用设置各种状态， 来管理应用

![主流程](/images/singleSpa/1.png)

接下来， 我们来看一下single-spa框架的主流程是

- 首先，将路由改变需要切换的app或者用户用户手动调起的app包装后， 统一push到事件队列里面

- 如果容器应用没有启动， 那么加载需要加载的app。加载成功之后调用finish方法， 清空当前的执行栈

- 如果容器已经启动，切换app的时候，卸载不需要的app，加载需要的app，挂载需要的app。 完成之后

  清空当前的执行栈。

- 如果过程中，有路由改变等事件， 需要切换app的时候， 将它放入到下一个执行栈中

## singleSpa中的状态机

我们先来熟悉一下single-spa中应用的各种状态

![各种状态](/images/singleSpa/state.png)

- 默认为NOT_LOAD
- LOAD:   开始加载应用
- NOTBOOTSTRAP:  加载成功未启动
- BOOTSTRAPING: 开始挂载/启动， 执行bootstrap生命周期， 只执行一次
- NOT_MOUNTED：BOOTSTRAP生命周期函数执行完成(成功)
- MOUNTING: 开始执行mount生命周期函数
- MOUNTED:  app挂载成功， mount生命周期函数执行成功， 可以执行Vue的$mount()或ReactDOM的render()
- UNMOUNTING: unmount生命周期函数执行，执行 Vue的$destory()或ReactDOM的unmountComponentAtNode()。
- SKIP_BECAUSE_BROKEN： 状态改变出错
- LOAD_ERROR： 一般是用户提供的信息有误，导致加载出错

```js
export const NOT_LOADED = 'NOT_LOADED';
export const SKIP_BECAUSE_BROKEN = 'SKIP_BECAUSE_BROKEN';
export const LOAD_ERROR = 'LOAD_ERROR'
export const LOAD_SOURCE_CODE = 'LOAD_SOURCE_CODE'
export const NOT_BOOTSTRAP = 'NOT_BOOTSTRAP'
export const BOOTSTRAPPING = 'BOOTSTRAPPING';
export const NOT_MOUNTED = 'NOT_MOUNTED';
export const MOUNTED = 'MOUNTED'
export const MOUNTING = 'MOUNTING';
export const UNMOUNTING = 'UNMOUNTING';
export const UPDATING = 'UPDATING';


// import { NOT_LOADED, notLoadError, notSkip, isntLoaded, shouldBeActivity } from './appsHelper';
export function notSkip(app) {
    return app.status !== SKIP_BECAUSE_BROKEN;
}

export function notLoadError(app) {
    return app.status !== LOAD_ERROR;
}

export function isntLoaded(app) {
    return app.status === NOT_LOADED;
}

export function isLoaded(app) {
    return app.status !== NOT_LOADED && app.status !== SKIP_BECAUSE_BROKEN && app.status !== LOAD_ERROR;
}

export function isActive(app) {
    return app.status === MOUNTED
}

export function isntActive(app) {
    return !isActive (app)
}

export function shouldBeActivity(app) {
    try {
        return app.activityWhen(window.location)
    }catch(e) {
        app.status = SKIP_BECAUSE_BROKEN
        console.log(e)
    }
}

export function shouldntBeActivity(app) {
    try {
        return !app.activityWhen(window.location);
    } catch(e) {
        app.status = SKIP_BECAUSE_BROKEN;
        throw e;
    }
}
```

- 需要加载的app

  ```js
  /**
   * @function 获取需要被加载的app
   * 当前app 的状态不是SKIP_BECAUSE_BROKEN, 不是LOAD_ERROR，是NOT_LOADED 同时是应该被加载
   */
  export function getAppsToLoad() {
      return APPS.filter(notSkip).filter(notLoadError).filter(isntLoaded).filter(shouldBeActivity)
  }
  
  
  /**
   * @function 获取需要可以被挂载的app
   * 当前app 的状态不是SKIP_BECAUSE_BROKEN, 已经加载过，没有被激活， 同时是应该挂载
   */
  export function getAppsTomount() {
      return APPS.filter(notSkip).filter(isLoaded).filter(isntActive).filter(shouldBeActivity)
  }
  
  /**
   * @function 获取需要待被挂载的app
   */
  export function getAppsToUnmount() {
      return APPS.filter(notSkip).filter(isActive).filter(shouldntBeActivity)
  }
  
  /**
   * @function 获取当前已经挂载的app
   */
  export function getMountedApps() {
      return APPS.filter(app => {
          return isActive(app)
      })
  }
  ```

## 整个流程

- 首次加载，用户通过registerApplication注册应用, 这里是将所有注册的app收集起来，后面会通过各种状态筛选需要的app。 最后调用invoke函数

  ```js
  /**
   * @function 注册App
   * @param {string} appName 要注册的app的名称
   * @param {Function: Promise | Object} loadFunction app异步加载函数， 或者app的内容
   * @param {Function: boolean} activityWhen  判断该app应该何时启动
   * @param {Object} customProps  自定义参数配置
   * return Promise
   */
  const APPS = []
  export function registerApplication(appName, loadFunction, activityWhen, customProps = {}) {
      // 判断参数是否合法
      if(!appName || typeof appName !== 'string') {
          throw new Error('appName不可以是一个空字符串')
      }
      if(!loadFunction) {
          throw new Error('loadFunction must be a function or object')
      }
      if(typeof loadFunction !== 'function') {
          loadFunction = () => Promise.resolve(loadFunction)
      }
      if(typeof activityWhen !== 'function') {
          throw new Error('activityWhen must be a function')
      }
  
      APPS.push({
          name: appName,
          loadFunction,
          activityWhen,
          customProps,
          status: NOT_LOADED
      })
  
      invoke();
  }
  ```

- `invoke函数主要就是控制应用的状态, 同时控制当前event-loop的队列。
  如果当前event-loop没有处理完成， 那么将需要被操作的app放入到下一次的event-loop中
  同时判断容器应用有没有被启动， 如果启动， 那么调用performAppChanges切换应用
  如果没有启动， 那么开始加载需要被加载的应用`

  ```js
  // 默认没有进行event-loop
  let appChanges = false;
  let changesQueue = [];
  
  /**
   * @function 控制应用的状态，同时控制当前event-loop的队列
   * @param {*} pendings 当前事件队列
   * @param {*} eventArgs 路由切换的参数
   */
  export function invoke(pendings = [], eventArgs) {
      // 先检查appChanges有没有在做event-loop， 如果正在做event-loop， 将当前app放入到事件队列里面等待
      // 事件队列不存储app的信息， 所有的app都会放入到APPS的全局状态中，每次先获取
      if(appChangesUnderway) {
          return new Promise((resolve, reject) => {
              changesQueue.push({
                  success: resolve,
                  failure: reject,
                  eventArgs
              })
          })
      }
  
      // 表示当前正在进行event-loop
      appChangesUnderway = true
  
      // 这里是整个SingleSpa的状态， 判断容器应用是否已启动
      if(isStarted()) {
          return performAppChanges()
      }
  
      // 获取需要被加载的app
      return loadApps();
  }
  ```

- performAppChanges， 切换应用状态

  ```js
    /**
     * @function 如果应用已经启动，那么卸载不需要的app，加载需要的app，挂载需要的app
    */
    function performAppChanges() {
        // 先卸载不需要的app， 这里是放到promise中， 开始执行
        let unmountPromise = getAppsToUnmount().map(toUnmountPromise);
        unmountPromise = Promise.all(unmountPromise);

        // will load app --> NOT_MOUNTED
        let loadApps = getAppsToLoad()
        loadApps = loadApps.map(app => {
            // 先去加载， 加载完成之后调用bootstrap, 然后卸载，然后加载
            return toLoadPromise(app).then((app) => {
                return toBootstrapPromise(app)
                    .then(() => unmountPromise)
                    .then(() =>  toMountPromise(app));
            });
        });

        // will mount app --> NOT_MOUNTED
        let mountApps = getAppsTomount();

        // 针对load和mount的app做去重， 这里看一下， 为什么要去重
        mountApps = mountApps.filter(app => loadApps.indexOf(app) === -1);

        mountApps = mountApps.map((app) => {
            return toBootstrapPromise(app)
                .then(() => unmountPromise)
                .then(() => toMountPromise(app))
        })

        // 卸载没有问题的时候， 进行挂载新的
        return unmountPromise.then(() => {
        let allPromises = loadApps.concat(mountApps);
        return Promise.all(allPromises.map(toMountPromise)).then(() => {
            callAllCaptureEvents();
            return finish();
        }, e => {
            // 当一个promise状态已经改变的时候， 再次调用的时候不会在改变
            pendings.forEach(item => item.failure(e));
            throw e;
        })
        }, e => {
            callAllCaptureEvents();
            console.log(e);
        });
    }
  ```

- 加载完应用或者切换应用完成的时候， 调用finish方法清空当前的事件队列， 开始调用下一次事件队列

  ```js
    function finish() {
        // 路由改变， 或者是调用start方法
        // /home, 正在加载appA的时候， route变为了/index
        // appA加载完成之后， 必须立马加载appB， 将appB放到changesQueue里面，
        let returnValue = getMountedApps();

        if(pendings.length) {
            pendings.forEach(item => item.success(returnValue))
        }

        // 当前的循环已经完成
        appChangesUnderway = false;
        if(changesQueue.length) {
            // backup就是当前循环中被推到事件队列里的事件， 这些事件交给下一次的invoke处理
            let backup = changesQueue;
            changesQueue = [];

            invoke(backup);
        }
        return returnValue
    }
  ```



singleSpa的主流程大概就是这样。接下来我们分析两个问题

1. singleSpa是如何实现不刷新就可以切换多框架的应用的。
2. singleSpa中都做了哪些优化

## singleSp中的路由管理

这里主要介绍一下singleSpa实现不刷新就可以切换多框架的应用的。
其实， vue或者react中的hash路由或者history路由，
都是基于浏览器的**hashchange**和**popstate**事件来做的(pushState, replaceState)， 
所以在singleSpa中， 对这几个事件进行了拦截。 当路由改变的时候， 
优先触发singleSpa的切换应用方法(invoke)， 
之后在执行Vue或者react等其他应用的路由方法

```js
import { invoke } from "./invoke";

const HIJACK_EVENTS_NAME = /^(hashchange|popstate)$/i;
const EVENT_POOL = {
    hashchange: [],
    popstate: []
}

function reroute() {
    invoke([], arguments)
}

window.addEventListener('hashchange', reroute);
window.addEventListener('popstate', reroute);

const originalAddEventListener = window.addEventListener;
const originalRemoveEventListener = window.removeEventListener;

// 只针对性的拦截:hashchange&popstate 这两种
window.addEventListener = function(eventName, handler, ) {
    if(eventName && HIJACK_EVENTS_NAME.test(this.eventName)) {
        EVENT_POOL[eventName].indexOf(handler) === -1 && EVENT_POOL[eventName].push(handler)
    }else {
        originalAddEventListener.apply(this, arguments)
    }
}

window.removeEventListener = function(eventName, handler, ) {
    if(eventName && HIJACK_EVENTS_NAME.test(this.eventName)) {
        let events = EVENT_POOL[eventName]
        events.indexOf(handler) > -1 && 
        (EVENT_POOL[eventName] = events.filter(fn => fn !== handler))
    }else {
        originalRemoveEventListener.apply(window , arguments)
    }
}

function mokePopStateEvent(state) {
    return new PopStateEvent('popstate', { state })
}

const originalPushState = window.history.pushState;
const originalReplaceState = window.history.replaceState;

window.history.pushState = function(state, title, url) {
    let result = originalPushState.apply(this, arguments);

    reroute(mokePopStateEvent(state));

    return result;
}

window.history.replaceState = function(state, title, url) {
    let result = originalReplaceState.apply(this, arguments);

    reroute(mokePopStateEvent(state));

    return result;
}

export function callCaptureEvents(eventsArgs) {
    if(!eventsArgs) {
        return
    }

    if(!Array.isArray(eventsArgs)) {
        eventsArgs = [eventsArgs]
    }

    let name = eventsArgs[0].history;

    if(!EVENT_POOL[name] || EVENT_POOL[name].length === 0) {
        return;
    }

    EVENT_POOL[name].forEach(handler => {
        handler.apply(null, eventsArgs)
    })
}
```

## Single-spa中还做了哪些优化

比如说应用超时处理等

singleSpa中对每个应用状态的切换都做了超时处理， 当应用切换超时的时候，可以自定义操作

```js

const TIMEOUTS = {
    bootstrap: {
        milliseconds: 3000,
        rejectWhenTimeout: false
    },
    mount: {
        milliseconds: 3000,
        rejectWhenTimeout: false
    },
    unmount: {
        milliseconds: 3000,
        rejectWhenTimeout: false
    }
}

export function reasonableTime(lifecyclePromise, description, timeout) {
    return new Promise((resolve, reject) => {
        let finished = false;
        lifecyclePromise.then((data) => {
            finished = true;
            resolve(data);
        }).catch(e => {
            finished = true;
            reject(e);
        })

        setTimeout(() => {
            if(finished) { return };
            if(timeout.rejectWhenTimeout) {
                reject(`${description}`);
            }else {
                console.log('timeout but waiting');
            }
        }, timeout.milliseconds);
    })
}

export function ensureTimeout(timeouts = {}) {
    return {
        ...TIMEOUTS,
        ...timeouts
    }
}
```

## 备注

vue和react切换的区别
Vue 2.x的dom挂载，采取的是 **覆盖Dom挂载** 的方式。例如，组件要挂载到`#app`上，那么它会用组件覆盖掉`#app`元素。
但是React/Angular不同，它们的挂载方式是在目标挂载元素的内部`添加元素`，而不是直接覆盖掉。 例如组件要挂载到`#app`上，那么他会在`#app`内部挂载组件，`#app`还存在。
这样就造成了一个问题，当我从 vue子项目 => react项目 => vue子项目时，就会找不到要挂载的dom元素，从而抛出错误,不过可以在vue手动拼`#app`。
