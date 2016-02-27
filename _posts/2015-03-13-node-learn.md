---
layout: post
title: node js 
category : [node, learn]
tags : [node, learn]

---
nodejs learn

### 1. nodejs 初步


    var http = require("http"); //加载nodejs的内置模块  
    http.createServer(function(req, res){  
    	res.writeHead(200, {"Content-Type" : "text/plain"});  
    	res.write("Hello World");  
    	res.end();  
    }).listen(8888);  


> 创建一个http server 在内置函数中处理响应的请求和响应


#### nodejs 文件

>server.js
 

    var url = require("url");
    var http = require("http");

    function start(route, handler) {

    function OnReq(req, res){
    var pathname = url.parse(req.url).pathname;
    route(pathname, handler, res);
    }

    http.createServer(OnReq).listen(8888);
    console.log("serrver start ")
    }

    exports.start = start;


> index.js 

    var server = require('./serrver');
    var route = require('./route');
    var handler = require('./handler');

    server.start(route.route, handler);


> route.js 


    function route(pathname, handler, res){

      console.log("route " + pathname);
      if(pathname == "/download"){
        console.log("333333");
        handler.download(res);
      }else if(pathname == "/upload"){
        console.log("1111111");
        handler.upload(res);
      }
    }

    exports.route = route;


> handler.js 


    function download(res){
      var body = '<html>'+
        '<head>'+
        '<meta http-equiv="Content-Type" content="text/html; '+
        'charset=UTF-8" />'+
        '</head>'+
        '<body>'+
        '<form action="/upload" method="post">'+
        '<textarea name="text" rows="20" cols="60"></textarea>'+
        '<input type="submit" value="Submit text" />'+
        '</form>'+
        '</body>'+
        '</html>';

      res.writeHead(200, {"Content-Type" : "text/html"});
      res.write(body);
      res.end();

      console.log("download function ");
    }

    function upload(res){
      res.writeHead(200, {"Content-Type" : "text/html"});
      res.write("hello upload");
      res.end();

      console.log("upload function");
    }

    exports.upload = upload;
    exports.download = download; 


### 2. express 使用

由于使用Rails的习惯，自己构建了类似Rails的使用目录结构：

> [wechats](https://github.com/Freeza91/wechat-shake-game)

> [huobi-auto](https://github.com/Freeza91/btc-auto-trade)


