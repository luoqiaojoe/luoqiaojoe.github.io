---
layout: post
author: 罗必才
title: 反向Ajax和socket.io基础
category: 理论知识
tag: [理论]
---

#### 反向Ajax

传统客户端与服务端的通信，都是客户端向服务端发起一个请求，然后服务端返回相应的数据。但在实际web应用中，存在一些搞不定的场景：
+ 在线咨询
+ 客服系统
+ 在线聊天室
+ 在线联机游戏

这些场景无一例外的，都需要客户端与服务端的双向通信，也就是，服务端也能主动向客户端发送数据，所以这就涉及到了反向Ajax。

**如何实现呢？**

以往的实现方式有：
+ http 轮询
+ Comet（长轮询）

**轮询，**就是每隔一个固定的时间，从浏览器端向服务端发起一个ajax请求，获取新的数据。
**长轮询，**就是客户端不停的向服务器发送请求以获取最新的数据信息。这里的“不停”其实是有停止的，只是我们人眼无法分辨是否停止，它只是一种快速的停下然后又立即开始连接而已。

这两种方式，共同的缺点都是:

+ 请求中有大半是无用的，大大的浪费带宽和服务器资源
++ 及时性不够，不能获取到最新的数据（状态）

**真正的解决方案，就是web socket。**

>Web socket 是 一个实现双向通信的协议。在HTML5中新增的一个特性，用来实现浏览器端和服务端的双向通信。

`Websocket`只是一个协议，就是一个规范。针对这对协议，有多种实现，大概有4、5种左右。

**其中，最好的就是socket.io。**
	
>Socket.io是一个完全由JavaScript实现、基于Node.js、支持WebSocket的协议用于实时通信、跨平台的开源库，它包括了客户端的JavaScript和服务器端的Node.js。

**Socket.io是基于js，分成两部分：**
+ 客户端js
+ 服务端的js

node.js作为服务端语言，有两大优点：
+ 实时性
+ 异步非阻塞 I/O

**其中的实时性，就体现在socket.io上。**

#### Socket.io的基本使用

一.安装
官网：https://socket.io/docs/

需要安装，分成两个方面：
+ 服务端的安装， npm install socket.io
+ 浏览器端的安装，下载一个societ.io.js文件

二.基本使用
在服务端，此处用Express作为例子：
{% highlight c %}
//首先，导入核心模块
const http = require('http');
const path = require('path');
const sio = require('socket.io');
const app = express();

//假设我们的静态资源文件都在public文件夹下，且我们的 html 文件名为 socke.io.html
//使用静态资源托管
app.use(express.static(path.join(__dirname,'public')));

//实现首页面路由
app.get('/',(req,res)=>{
	res.sendFile(path.join(__dirname,'socket.io.html'))
})

//必须使用server来启动http服务
const server = http.createServer(app);//server 是http创建的，但是基于app的

//获取一个io对象，将sio应用到server上
const io = sio.listen(server)

//需要使用一个io对象，去建立一个socket的链接
io.on('connection',(socket) => {
	//其中socket对象就是发送请求和相应请求的对象
	console.log('xixixi')
})

server.listen('3000',() => {
	console.log('http server is listening in port 3000...')
})
{% endhighlight %}

对应的，浏览器端的代码：
{% highlight c %}
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Socket.io的基本使用</title>
	<script src=/js/socket.io.js></script>
</head>
<body>
	<h2>socket.io使用</h2>
	<script>
		//一旦引入了socket.io.js文件，就提供了一个io对象
		var socket = io.connect('http://localhost:3000');
		console.log(socket);
	</script>
</body>
</html>
{% endhighlight %}

我们就可以分别查看，浏览器控制台，和你的开启本地服务器的小黑窗，会有相应的结果被打印出来。

这说明，浏览器端和服务端的`socket`通道已经建立好了，这就是用来实现**双向通信**的通道。

在浏览器和服务端任何一方，**都可以**使用socket的如下方法，来发请求或响应请求
+ emit，发起请求事件
+ on，注册时间，就可以响应

还有一个特殊的，就是`broadcast`，在服务端向所有的客户端广播事件。

我们可以实现一个简单的，浏览器和服务器的聊天过程：

##### 第一步，在浏览器端，通过`socket`对象发射一个事件，如下：

{% highlight c %}
//我们在浏览器端的代码中加上一个输入框和按钮，再引入jQuery

<input type="text" id="msg">
<button id="btn">发送</button>

<script>
	//一旦引入了socket.io.js文件，就提供了一个io对象
		var socket = io.connect('http://localhost:3000');
		console.log(socket);
		$('#btn').on('click',function () {
			// 向服务端发射一个事件，并将输入的值传递过去
			socket.emit('publish',$('#msg').val())
		})
</script>
{% endhighlight %}

##### 第二步，需要在服务端，注册publish事件。

{% highlight c %}
//需要使用一个io对象，去建立一个socket的链接
io.on('connection',(socket) => {
	//其中socket对象就是发送请求和相应请求的对象
	console.log('xixixi');
	
	//注册publish事件
	socket.on('publish',(data) => {
		console.log('来自浏览器的消息：'+ data)
	})
})
{% endhighlight %}

相应的，服务端也需要向客户端发送请求。

##### 第三步，在服务端，向客户端发射一个事件，如下:

{% highlight c %}
//需要使用一个io对象，去建立一个socket的链接
io.on('connection',(socket) => {
	//其中socket对象就是发送请求和相应请求的对象
	console.log('xixixi');
	
	//注册publish事件
	socket.on('publish',(data) => {
		console.log('来自浏览器的消息：'+ data);

		//发射一个reply事件
		socket.emit('reply','辛苦了！')
	})
})
{% endhighlight %}

##### 第四步，在浏览器端，需要注册reply事件。

{% highlight c %}
//我们在浏览器端的代码中加上一个输入框和按钮，再引入jQuery

<input type="text" id="msg">
<button id="btn">发送</button>

<script>
	//一旦引入了socket.io.js文件，就提供了一个io对象
	var socket = io.connect('http://localhost:3000');
	console.log(socket);
	$('#btn').on('click',function () {
		// 向服务端发射一个事件，并将输入的值传递过去
		socket.emit('publish',$('#msg').val())
	});

	//注册reply事件
	socket.on('reply',function(data){
		console.log('来自服务端的问候：'+ data)
	})
</script>
{% endhighlight %}

同样的，我们就可以再浏览器的控制台，和电脑的小黑窗里看到相应的消息被打印出来。


所有代码如下（基于Express）：

**服务端**

{% highlight c %}
//首先，导入核心模块
const http = require('http');
const path = require('path');
const sio = require('socket.io');
const app = express();

//假设我们的静态资源文件都在public文件夹下，且我们的 html 文件名为 socke.io.html
//使用静态资源托管
app.use(express.static(path.join(__dirname,'public')));

//实现首页面路由
app.get('/',(req,res)=>{
	res.sendFile(path.join(__dirname,'socket.io.html'))
})

//必须使用server来启动http服务
const server = http.createServer(app);//server 是http创建的，但是基于app的

//获取一个io对象，将sio应用到server上
const io = sio.listen(server)

//需要使用一个io对象，去建立一个socket的链接
io.on('connection',(socket) => {
	//其中socket对象就是发送请求和相应请求的对象
	console.log('xixixi');
	
	//注册publish事件
	socket.on('publish',(data) => {
		console.log('来自浏览器的消息：'+ data);

		//发射一个reply事件
		socket.emit('reply','辛苦了！')
	})
})

server.listen('3000',() => {
	console.log('http server is listening in port 3000...')
})
{% endhighlight %}


**浏览器端**

{% highlight c %}
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Socket.io的基本使用</title>
	<script src=/js/socket.io.js></script>
</head>
<body>
	<h2>socket.io使用</h2>
	<script>
		//一旦引入了socket.io.js文件，就提供了一个io对象
		var socket = io.connect('http://localhost:3000');
		console.log(socket);
		$('#btn').on('click',function () {
			// 向服务端发射一个事件，并将输入的值传递过去
			socket.emit('publish',$('#msg').val())
		});

		//注册reply事件
		socket.on('reply',function(data){
			console.log('来自服务端的问候：'+ data)
		})
	</script>
</body>
</html>
{% endhighlight %}



