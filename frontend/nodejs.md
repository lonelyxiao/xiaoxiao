# 前言

js想要实现复杂的操作，必须依赖宿主环境的api， 例如：浏览器/操作系统

# js基础

- 定义带函数的对象

```javascript
<script>
    var obj={
        a1:'哈哈哈',
        a2:function(a){
            alert(a);
        }
    }
    console.log(obj.a2(111));
</script>
```

- 将函数作为函数传递

```javascript
function f1(a){
    a(1,2);
}
function f2(a,b) {
    console.log(a+b);
}
f1(f2);
```

- 闭包

f2运行完返回f3,由于作用域链问题，导致f1执行完后，不能销毁变量，形成闭包，这里，f3输出3，4

```javascript
var a=1
function f2() {
    a=2;
    function f3() {
        console.log(++a);
    }
    return f3;
}
f=f2();
f();
f();
```

# node入门

- node是什么

node是基于chorme v8引擎的JavaScript运行环境

他是除了浏览器之外的，可以让JavaScript运行的环境

- node能做什么
  - web服务器
  - 命令行工具
  - 网络爬虫
  - 桌面应用程序开发（<http://www.electronjs.org/>）

## node运行js代码

```shell
E:\code\nodejs\20200409>node 1.test.js
1
```

## 启动一个http服务

```javascript
//引入node的http模块
const http = require('http');
//创建服务
const server = http.createServer();

server.on('request', function(res, rs){
    //http接受到的数据都放在第一个参数中
    console.log(res);
    //返回客户端数据
    //设置返回数据解析编码
    rs.setHeader('Content-Type', 'text/plain;charset=utf-8');
    rs.write('哈哈哈');
    rs.write('hello');
    rs.end();
});
//启动监听
server.listen(8080, function(){
    //当服务启动之后，执行该函数
    console.log('127.0.0.1：8080 服务已启动');
})
```

# 第三方模块

## 第三方处理时间模块

```shell
E:\code\nodejs>npm i moment
```

目录中多了个package-lock.json文件夹，里面是moment模块的信息

引入使用

```javascript
const moment = require('moment');
console.log(moment(new Date()).format('YYYY-MM-DD HH:mm:ss'))
```

## NPM

它是nodejs自带的包模块管理工具，可以帮助我们快速安装和管理依赖包

- 修改nmp镜像地址

```shell

E:\code\nodejs>npm config list
; cli configs
metrics-registry = "https://registry.npmjs.org/"
E:\code\nodejs>npm config set registry=https://registry.npm.taobao.org
```

- 初始化项目

初始化项目之后，会产生产生package.json文件，这个文件管理整个项目的模块，在另一个项目下载本项目需要的依赖时，只需要npm install即可从这个项目读取对应的下载信息进行下载相关模块

```shell
E:\code\nodejs>npm init
```

# node自动重启

全局安装nodeme

```shell
E:\code\nodejs>npm i nodemon -g
```

启动项目

```shell
E:\code\nodejs\20200412>nodemon moment.1.js
```

# commonjs规范

服务器端规范

## 导出模块

文件名就是导出的模块名

- 第一种写法

新建一个文件mode.js

```javascript
var tfun = function(a, b){
    return a+b;
}
var c=2;

exports.tfun=tfun;
exports.c=c;	
```

将导出的模块引入

```javascript
const mode = require('./mode');
console.log("tfun:"+mode.tfun(1,2));
console.log("c:"+mode.c);
```

- 第二种写法

```javascript
module.exports=function(b,c){
    return b*c
};
```

```javascript
console.log("fun:"+mode(1,3));
```

## 第三方加载规则

如：const moment = require('moment');

去核心模块找，然后去node_modules（去同级目录，或者往上层目录找）中找，然后找到这个名字的文件夹（moment）

# 爬取网页图片

- 安装网页请求与dom解析模块

```shell
E:\code\nodejs>npm i request cheerio
```

- 代码

```javascript
//爬取地址
const url = 'http://photo.sina.com.cn/';

const path = require('path');
//保存地址
const imgPath = 'E://code//nodejs//20200413//img';

const request = require('request');
const fs = require('fs');
const cheerio = require('cheerio');

function start(){
    //抓取网页
    request(url, function(err, res, body){
        console.log('开始解析网页');
        if(!err && res){
            findImage(body, downLoad);
        }
    })
}
function findImage(body, callback){
    let $ = cheerio.load(body);
    $('img').each(function(i, elem){
        let img = $(this).attr('src');
        console.log(img);
        callback(img, i);
    });
}
//下载第i张图片
function downLoad(img, i){
    //获取图片后缀
    let ext =img.split('.').pop();
    //下载图片
    try{
        request(img).pipe(fs.createWriteStream(path.join(imgPath, i+'.'+ext), {
            'encoding':'utf-8'
        }))
    }catch(e){
        //console.log(e)
    }
    
}

start();
```



# ES6

## let和const

- 块作用域
- 不允许重复定义
- let变量，const常量

# pm2守护进程方式启动

```shell
pm2 start index.js
```

# koa web框架

# electronjs

安装

```shell
cnpm i -g electron@^8.2.2
```



## demo

建立一个目录，新建index.js index.html文件

输入npm init命令初始化项目

修改package.json文件

```json
"main": "index.js",
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "electron ."
},
```

index.js

```javascript
const { app, BrowserWindow } = require('electron')

function createWindow () {   
  // 创建浏览器窗口
  let win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true
    }
  })

  // 加载index.html文件
  win.loadFile('index.html')
}

app.whenReady().then(createWindow)
```

index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Hello World!</title>
    <!-- https://electronjs.org/docs/tutorial/security#csp-meta-tag -->
    <meta http-equiv="Content-Security-Policy" content="script-src 'self' 'unsafe-inline';" />
  </head>
  <body>
    <h1>Hello World!</h1>
    We are using node <script>document.write(process.versions.node)</script>,
    Chrome <script>document.write(process.versions.chrome)</script>,
    and Electron <script>document.write(process.versions.electron)</script>.
  </body>
</html>
```

启动项目

```shell
npm start
```

