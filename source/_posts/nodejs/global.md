---
title: 全局对象
abbrlink: b585e161
date: 2020-06-25 13:40:50
keywords: nodejs
tags: [nodejs]
---

1. 这些对象在所有模块中都可用。 以下变量可能看起来是全局的，但实际上不是。 它们只存在于模块的作用域中，参见模块系统文档：

 * global == window 全局对象
 * module 当前模块对象[loaded、children]
 * exports 导出对象[module.exports别名, 但exports不能直接赋值后导出]
 * require 加载模块的方法[cache, resovle,]
 * __dirname当前模块所在的目录的绝对路径
 * __filename 当前模块的绝对路径
nodejs中每个node模块就是独立模块作用域; 
2. 定时器相关 // 同步 > nextTick > setTimeout|setInterval > setImmediate > 异步IO;
3. console、Event、global;
```nodejs
// 订阅发布
function Event(name){
    this.name = name;
    this._events = {};

    //注册监听
    this.on = function(eventName,callback){
        if(this._events[eventName]){//有人已经订阅过了这个事件
            this._events[eventName].push(callback);
        }else{
            this._events[eventName] = [callback];
        }
    }
    this.emit = function(eventName){
        var args = Array.prototype.slice.call(arguments,1);
        var callbacks = this._events[eventName];
        var self = this;
        callbacks.forEach(function(callback){
            callback.apply(self,args);
        });
    }
    this.removeListener = function(eventName,callback){
        this._events[eventName] = this._events[eventName].filter(function(cb){
        return cb != callback;
    });
    }
    this.once = function(eventName,callback) {
        function onceCallback(){
            callback.apply(this,arguments);
            this.removeListener(eventName,onceCallback);
        }
        this.on(eventName,onceCallback);
    }
}
```

```nodejs
process.on('uncaughtException',function(e){
    console.log('uncaughtException', e.message);
});

//function x(module,exports,require,__dirname,__filename){
  //console.trace();//显示当前的调用当前堆栈
//}

console.log(global);
process.stdout.write('hello'); // 输出到控制台
console.log(process.pid, process);

process.stdin.on('data',function(data){ // 控制台输入
 console.log(data.toString());
});

process.on('exit',function(){
    console.log('退出前执行');
});

// ctrl + c 发送一个信号,已确保能正常出发exit事件;
process.on('SIGINT', function() {
    console.log('SIGINT');
    process.exit();
});

// process.argv
// 中断进程pid
process.kill(process.pid);
process.chdir('4.global');
console.log(process.cwd()); // 改变当前目录, cwd也会对应的变化;
console.log(__dirname); // __dirname 固定为当前工作目录,不会更改;
```

1. Buffer 用于处理二进制数据;string_decoder
2. MessageChannel、MessageEvent、MessagePort、performance;
3. process、queueMicrotask;
4. URL、URLSearchParams、WebAssembly;

## 调试node - chrome
node --inspect --inspect-brk x.js
json res.end(`${cb}(${data})`);
cors: writeHead(200, {
    'Access-Control-Allow-Origin: *'
});
middleware cors: 
cheerio|puppter|events
fs.watch|rename|readDir|stat|watchFile
readline: 行读取器
crypto: 加密

## stream
```nodejs
const fs = require('fs');
const zlib = require('zlib');
const gzip = zlib.crateGzip();

const rs = fs.createReadStream(fileInPath);
const ws = fs.createReadStream(fileOutPath);

rs.pipe(zip).pipe(ws);
// rs.pipe(ws);
// sw.write(rs);
```

```nodejs
const crypto = require('crypto');
const pwd = crypto.createHash('md5|sh1').update(pwd).digest('hex');

```
