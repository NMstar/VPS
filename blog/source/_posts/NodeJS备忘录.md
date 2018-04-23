---
title: NodeJS备忘录
tags: [Node JS]
---
## 语法记录
``` js
// 导入资源包:
var http = require("http");

// 终端打印:
console.log("Some Thing");

// 事件循环:

// 引入events模块:
var events = require('events');
// 创建 eventEmitter对象:
var eventEmitter = new events.EventEmitter();
// 绑定事件处理程序:
eventEmitter.on('eventName',eventHandler);
// 触发事件
eventEmitter.emit('eventName');

// 定时器
// 1000 ms 执行前一个函数
setTimeout(function(){
    eventEmitter.emit('eventName')
}, 1000);

```

## 注意
1. 一个event可以注册多个回调
2. event可以删除回调