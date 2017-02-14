---
title: web跨域的几种实现方式
date: 2016-08-10 19:50:48
comments: true
reward: true
tags:
    - 跨域
    - josnp
    - withCredentials
    - 代理
    - javascript
---
今天在为我们系统接入阿里域登录的时候遇到了跨域问题，以前在系统中都是后端设置允许跨域或者采用jsonp来实现跨域。这次从web端角度对常见的跨域问题进行总结，防止以后再遇到类似问题。

同源策略/SOP（Same origin policy）是一种约定，它是浏览器最核心也最基本的安全功能，如果缺少了同源策略，浏览器很容易受到XSS、CSFR等攻击

SOP要求两个通讯地址的*协议、域名、端口号*必须相同，否则两个地址的通讯将被浏览器视为不安全的，并被block下来。比如“http页面”和“https页面”属于不同协议；“qq.com”、“www.qq.com”、“a.qq.com”都属于不同域名（或主机）；“a.com”和“a.com:8000”属于不同端口号。这三种情况常规都是无法直接进行通讯的。

![](http://i.imgur.com/PK8wdZF.png)
<!-- more -->
模拟不同源的实现可以采用iframe来实现：
```
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>模拟跨域</title>
</head>
<body>
<iframe src="http://baidu.com"></iframe>
<script>
    window.frames[0].onload = function () {
        alert("1");
    }
</script>
</body>
</html>
```
上述代码在chrome中报错
```
Uncaught DOMException: Blocked a frame with origin "https://www.baidu.com" from accessing a cross-origin frame.(…)
```
即我们无法监听百度首页文档onload的事件，因为top窗口跟iframe窗体是不同源的。

#目录  #
	
	- 服务端设置CORS
	- XDR
	- withCredentials
	- HTML5解决方案
		1. Cross-document messaging
		2. WebSocket
	- JSONP	
	- iframe形式
		1. document.domain
		2. location.hash
		3. window.name
	- 其它形式
		1. 服务器代理
		2. flash socket

# 服务端设置CORS #
同域安全策略CORS（Cross-Origin Resource Sharing）是W3C在05年提出的跨域资源请求机制，它要求当前域（常规为存放资源的服务器）在响应报头添加Access-Control-Allow-Origin标签，从而允许指定域的站点访问当前域上的资源。
服务端(node)
```javascript
var http = require("http");
require("http").createServer(function(req,res){
    //报头添加Access-Control-Allow-Origin标签，值为特定的URL或“*”
    //“*”表示允许所有域访问当前域
    res.setHeader("Access-Control-Allow-Origin","*");
    res.end("OK");
}).listen(1234);
```
客户端
```
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>CORS</title>
    <script src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
</head>
<body>
<div>catching data...</div>
<script>
    $.ajax({
        url:"http://127.0.0.1:1234/",
        success:function(data){
            $("div").text(data)
        }
    })
</script>
</body>
</html>
```
结果：
运行客户端页面后，便能看到div内容成功变为服务端发来的“OK”，实现了两个不同域的页面间的通讯。通过上述代码我们也发现，CORS主要是在服务端上的实现（也不外乎是添加一个报头标签），客户端的实现跟常规的请求没啥出入。

不过CORS默认只支持GET/POST这两种http请求类型，如果要开启PUT/DELETE之类的方式，需要在服务端在添加一个"Access-Control-Allow-Methods"报头标签：
```
require("http").createServer(function(req,res){
  res.setHeader("Access-Control-Allow-Origin","http://127.0.0.1");
  res.setHeader(
    "Access-Control-Allow-Methods",
    "PUT, GET, POST, DELETE, HEAD, PATCH"
  );
  res.end(req.method+" "+req.url);
}).listen(1234);
```
# XDR #
恼人的IE8-是不支持上述的CORS滴，不过不走寻常路的巨硬在IE8开始引入了XDR(XDomainRequest)新特性（IE11已经不再支持该特性），它实现了CORS的部分规范，只支持GET/POST形式的请求。另外在协议部分只支持 http 和 https 。

在服务器端，依旧要求在响应报头添加"Access-Control-Allow-Methods"标签（这点跟CORS一致）。

在客户端，DR对象的使用方法与XHR对象非常相似，也是创建一个XDomainRequest的实例，调用open()方法，再调用send()方法。但与XHR对象的open()方法不同，XDR对象的open()方法只接收两个参数：请求的类型和URL，因为所有XDR请求都是异步执行的，不能用它来创建同步请求。

请求返回之后，会触发load事件，相应的数据也会保存在responseText属性中，如下所示：
```
var xdr = new XDomainRequest();
xdr.onload = function() {
    alert(xdr.responseText);
};
xdr.onerror = function() {
    alert("一个错误发生了！");
};
xdr.open("get", "http://127.0.0.1:1234/");
xdr.send(null);
```
由于XDR实在太过时，这里不做太多介绍，了解下即可

# withCredentials #
通过上述两种跨域介绍我们可以实现跨浏览器的跨域实现。withCredentials这种在请求头增加该属性的方法也是我在这次的项目中采用的方法。因为接入阿里登录系统其实是通过获取同域名的前一个重定向请求的cookie来发送跨域请求获得用户登录信息。
```javascript
function createCORSRequest(method, url) {
    var xhr = new XMLHttpRequest();
    if ("withCredentials" in xhr) {
        // "withCredentials"属性是XMLHTTPRequest2中独有的
        xhr.open(method, url, true);
    } else if (typeof XDomainRequest != "undefined") {
        // 检测是否XDomainRequest可用
        xhr = new XDomainRequest();
        xhr.open(method, url);
    } else {
        // 看起来CORS根本不被支持
        xhr = null;
    }
    return xhr;
}
var xhr = createCORSRequest('GET', url);
if (!xhr) {
    throw new Error('CORS not supported');
}
```
标准的CORS请求不对cookies做任何事情，既不发送也不改变。如果希望改变这一情况，就需要将withCredentials设置为true。

另外，服务端在处理这一请求时，也需要将Access-Control-Allow-Credentials设置为true。这一点我们稍后来说。

withCredentials属性使得请求包含了远程域的所有cookies，但值得注意的是，这些cookies仍旧遵守“同域”的准则，因此从代码上你并不能从document.cookies或者回应HTTP头当中进行读取。

![](http://i.imgur.com/UHittb2.png)


# HTML5解决方案 #
## 1. Cross-document messaging ##
在 Cross-document messaging 中，我们可以使用 postMessage 方法和 onmessage 事件来实现不同域之间的通信，其中postMessage用于实时向接收信息的页面发送消息，其语法为：
```
otherWindow.postMessage(message, targetOrigin);
```
```
otherWindow: 对接收信息页面的window的引用。可以是页面中iframe的contentWindow属性；window.open的返回值；通过name或下标从window.frames取到的值。
message: 所要发送的数据，string类型。
targetOrigin: 允许通信的域的url，“*”表示不作限制。
```

我们可以在父页面中嵌入不同域的子页面（iframe实现，而且常规会把它隐藏掉），在子页面调用 postMessage 方法向父页面发送数据：
父页面a.html
```
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>postMessage</title>
</head>
<body>
<iframe style="display:none;" id="ifr" src="http://127.0.0.1:63342/cross/b.html"></iframe>
<script type="text/javascript">
    window.addEventListener('message', function(event){
        // 通过origin属性判断消息来源地址
        if (event.origin == 'http://127.0.0.1:63342') {
            alert(event.data);    // 弹出从子页面post过来的信息
        }
    }, false);
</script>
</body>
</html>
```
子页面b.html
```
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>子页面</title>
</head>
<body>
<script type="text/javascript">
    var ifr = window.parent;  //获取父窗体
    var targetOrigin = 'http://localhost:63342';  // 若写成 http://127.0.0.1:10847 则将无法执行postMessage
    ifr.postMessage('这是传递给a.html的信息', targetOrigin);
</script>
</body>
</html>
```
## 2. WebSocket ##
web sockets是一种浏览器的API，它的目标是在一个单独的持久连接上提供全双工、双向通信。(同源策略对web sockets不适用)

web sockets原理：在JS创建了web socket之后，会有一个HTTP请求发送到浏览器以发起连接。取得服务器响应后，建立的连接会使用HTTP升级从HTTP协议交换为web sockt协议。

只有在支持web socket协议的服务器上才能正常工作。
```javascript
var socket = new WebSockt('ws://www.baidu.com');//http->ws; https->wss
socket.send('hello WebSockt');
socket.onmessage = function(event){
    var data = event.data;
}
```
# JSONP #
JSONP包含两部分：回调函数和数据。
回调函数是当响应到来时要放在当前页面被调用的函数。
数据就是传入回调函数中的json数据，也就是回调函数的参数了。
```javascript
function handleResponse(response){
    console.log('The responsed data is: '+response.data);
}
var script = document.createElement('script');
script.src = 'http://www.baidu.com/json/?callback=handleResponse';
document.body.insertBefore(script, document.body.firstChild);
/*handleResonse({"data": "zhe"})*/
//原理如下：
//当我们通过script标签请求时
//后台就会根据相应的参数(json,handleResponse)
//来生成相应的json数据(handleResponse({"data": "zhe"}))
//最后这个返回的json数据(代码)就会被放在当前js文件中被执行
//至此跨域通信完成
```
jsonp虽然很简单，但是有如下缺点：
1）安全问题(请求代码中可能存在安全隐患)
2）要确定jsonp请求是否失败并不容易

JSONP(JSON with Padding)是JSON的一种“使用模式”，主要是利用script标签不受同源策略限制的特性，向跨域的服务器请求并返回一段JSON数据。

常规前后端会约定好某个JSONP请求的callback名（比如随便起个名字“abc”），服务端返回的JSON数据会被这个callback名包裹起来，进而方便服务器区分收到的请求，也方便客户端区分其收到的响应数据。我们可以利用jQuery轻松实现JSONP：
客户端
```
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>JSONP</title>
    <script src="http://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>
</head>
<body>
<div></div>
<script>
    $.ajax({
        url:'http://127.0.0.1:1234/',
        dataType:"jsonp", //告知jQ我们走的JSONP形式
        jsonpCallback:"abc", //callback名
        success:function(data){
            console.log(data)
        }
    });
</script>
</body>
</html>
```
服务端（访问地址http://127.0.0.1:1234/ ）：
```javacript
/**
 * Created by leo.wz on 2016/8/10.
 */
var http = require('http');
var urllib = require('url');

var data = {'name': 'vajoy', 'addr': 'shenzhen'};

http.createServer(function(req, res){
    res.writeHead(200, { 'Content-type': 'text/plain'});
    var params = urllib.parse(req.url, true);
    //console.log(params);
    if (params.query && params.query.callback) {
        //console.log(params.query.callback);
        var str =  params.query.callback + '(' + JSON.stringify(data) + ')';//jsonp
        res.end(str);
    } else {
        res.end(JSON.stringify(data));//普通的json
    }
}).listen(1234)
```
客户端执行结果：
![](http://i.imgur.com/1fHlDnm.png)

不过JSONP始终是无状态连接，不能获悉连接状态和错误事件，而且只能走GET的形式。
# iframe形式 #
在很久以前的石器时代，对于不支持 XMLHttpRequest 的浏览器的最佳回溯方法之一就是使用IFRAME对象，当然常规只是用它来实现流模式的Comet，而不是解决跨域通信的问题。

使用iframe跨域其实有点剑走偏锋的既视感，也存在一些限制性。下面均来介绍下。
## 1. document.domain ##
该方法只适合主域相同但子域不同的情况，比如 www.a.com 和 script.a.com，我们只需要给这两个页面都加上一句 document.domain = 'a.com' ，就可以在其中一个页面嵌套另一个页面，然后进行窗体间的交互。
1) 在www.a.com/a.html中：
```javascript
document.domain = 'a.com';
var ifr = document.createElement('iframe');
ifr.src = 'http://www.script.a.com/b.html';
ifr.display = none;
document.body.appendChild(ifr);
ifr.onload = function(){
    var doc = ifr.contentDocument || ifr.contentWindow.document;
    //在这里操作doc，也就是b.html
    ifr.onload = null;
};
```
2) 在www.script.a.com/b.html中：
```javascript
document.domain = 'a.com';
```
这样在a的页面中就可以修改页面b的数据

## 2. location.hash ##
location.hash/url hash 是个好东西，在之前我们曾利用avalon前端路由来实现简单的SPA页面（这篇文章），便是助力于location.hash。

利用url地址改变但不刷新页面的特性（在url： http://a.com#hello 中的 '#hello' 就是location.hash，改变hash并不会导致页面刷新，所以可以利用hash值来进行数据传递）和iframe，我们可以实现跨域传递简单信息。

不过这个实现略麻烦，常规我们会想，在a.html下嵌套一个不同域的b.html，然后 a 和 b 互相修改彼此的hash值，也不断监听自己的hash值，从而实现我们的需求。可惜的是，大部分浏览器不允许修改不同域的父窗体的hash值（parent.location.hash），也就是说a虽能修改b的hash值，但反过来由b修改a的hash值却不成立。

为了解除该限制，我们可以在b页面中增加一个和a同域的iframe（c.html）来做代理，这样b可以修改c，而c可以修改a（即修改parent.parent.location.hash，别忘了a和c同域哦）。下面直接模拟这三个页面，做到让b向a传输信息（当然本质上是b向c，c再向a传输）：

该方法优点是兼容较好，缺点却显而易见——可传递的数据类型、长度均受限，数据还是直接显示在url上的，不够安全。另外其实现也较麻烦，还要搞setInterval不断监听，跟轮询没区别了。

## 3. window.name ##
window.name 的美妙之处在于，窗体的name值在页面跳转后依旧存在、保持原值（即使跳转的页面不同域），并且可以支持非常长的 name 值（2MB）。

# 其它形式 #
## 1. 服务器代理 ##
页面直接向同域的服务端发请求，服务端进行跨域处理或爬虫后，再把数据返回给客户端页面。依旧用node/iojs来模拟服务端
客户端：
```
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="initial-scale=1, maximum-scale=1,user-scalable=no">
    <title>proxy_test</title>

    <script>
      var f = function(data){
        alert(data.name);
      }
      var xhr = new XMLHttpRequest();
      xhr.onload = function(){
        alert(xhr.responseText);
      };
      xhr.open('POST', 'http://localhost:8888/proxy?http://geocode.arcgis.com/arcgis/rest/services/World/GeocodeServer', true);
      xhr.send("f=json");
    </script>
  </head>
  
  <body>
  </body>
</html>
```
服务端：
```javascript
var proxyUrl = "";
      if (req.url.indexOf('?') > -1) {
          proxyUrl = req.url.substr(req.url.indexOf('?') + 1);
          console.log(proxyUrl);
      }
      if (req.method === 'GET') {
          request.get(proxyUrl).pipe(res);
      } else if (req.method === 'POST') {
          var post = '';     //定义了一个post变量，用于暂存请求体的信息

        req.on('data', function(chunk){    //通过req的data事件监听函数，每当接受到请求体的数据，就累加到post变量中
            post += chunk;
        });
    
        req.on('end', function(){    //在end事件触发后，通过querystring.parse将post解析为真正的POST请求格式，然后向客户端返回。
            post = qs.parse(post);
            request({
                      method: 'POST',
                      url: proxyUrl,
                      form: post
                  }).pipe(res);
        });
      }
```

## 2. flash socket ##
其实在前面介绍socket.io的时候就有提到，在不兼容WebSocket的浏览器下，socket.io会以flash socket或Comet的形式来兼容，而flash socket是支持跨域通信的形式，跟WebSocket一样走的TCP/IP套接字协议。具体的实现可参考Adobe官方文档，本文不赘述。

# 参考文献 #
1.[http://www.html5rocks.com/en/tutorials/cors/](http://www.html5rocks.com/en/tutorials/cors/ "http://www.html5rocks.com/en/tutorials/cors/")
2.[http://www.cnblogs.com/vajoy/p/4295825.html](http://www.cnblogs.com/vajoy/p/4295825.html "浅谈WEB跨域的实现（前端向）")
3.[http://newhtml.net/using-cors/](http://newhtml.net/using-cors/ "http://newhtml.net/using-cors/")