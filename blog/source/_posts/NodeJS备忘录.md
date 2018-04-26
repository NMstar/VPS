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

// 构造Buffer
const buf = Buffer.from('runoob','ascii');

// 进行文件追加
var fs = require('fs');
var read = fs.createReadStream('../data/input.txt');
//设置第二个参数append
var write = fs.createWriteStream('../data/out.txt', { 'flags': 'a' });
//管道流读写操作
read.pipe(write);
console.log('执行完毕');

__filename 当前文件名
__dirname  当前目录

// util 工具
// 继承
util.inherits(Sub, Base); 

// 继承只会继承 Base.prototype 中定义的变量或者函数

// 将任意对象转化为字符串，比toString()显示更多的信息，常用于调试
util.inspect

// 判断是否是array
util.isArray(object)

// 判断是否是正则表达式
util.isRegExp(object)

// 判断是否是日期
util.isDate(object)

// 判断是否是错误
util.isError(object) 

```

## 注意
1. 一个event可以注册多个回调
2. event可以删除回调
3. Buffer是NodeJS的二进制类型