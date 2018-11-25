WebSocket
提供网页浏览器客户端与服务器的双向通讯和实时数据交换，可用于多角色的网页游戏，合作编程和代码共享，新闻或体育网站的实时字幕，在线画布，实时多用户的任务管理程序。这里用WS创建一个基于NodeJS的在线聊天室软件，并提供文件共享功能。

第一部分

环境搭建：
1. 编程环境：Atom
2. 命令行工具：cmder 在Windows系统下使用
3. 编程语言：NodeJS

初始化项目
1. [cmder]查看NodeJS版本：node -v
2. [cmder]创建本地项目目标文件夹：mkdir demo-websocket
3. [cmder]初始化项目：npm init
4. [cmder]安装Web服务器组件：nom install express --save
5. [cmder]安装Web服务器自动刷新组件：npm install -g nodemon
6. [Atom]项目根目录创建index.js文件： 
```
var express = require(‘express’); 
var app = express(); 
var server = app.listen(5000, function(){ 
  console.log(‘listening to requests  on port 5000’); 
}); 
```
7. [cmder]运行Web服务器：nodemon index
8. [Atom]在index.js中添加代码关联静态网页HTML文件： 
```
 app.use(express.static(‘public’)); 
```
9. [cmder]项目根目录创建public/index.html，输入HTML按下Tab，修改自动生成的代码： 
```
<title>Demo WebSocket</title> 
```
body部分添加 
```
 <h2>Welcome to WebSocket ChatApp</h2> 
```
10. 网页浏览器中打开 http://localhost:5000/ 验证初始化完成。
11. [Atom]创建样式定义public/styles.css，粘贴来自如下链接的代码： https://github.com/iamshaunjp/websockets-playlist/blob/lesson-2/public/styles.css
12. [Atom]在public/index.html的title部分下面引用样式定义： 
```
<link href="/styles.css" rel="stylesheet" /> 
```
13. 刷新网页 http://localhost:5000/ 验证样式得到了应用。

第二部分

安装和测试socket.io库
1. [cmder]Web服务器端安装：nom install socket.io --save
2. [Atom]更新根目录下index.js文件：
a. 头部添加引用：
```
 var socket = require('socket.io');
```
b. 底部添加调用： 
```
var io = socket(server); 
io.on('connection',function(socket){ 
  console.log('made socket connection',socket.id) };
);
```
3. 到 https://cdnjs.com/libraries/socket.io 拷贝类似如下链接备用：
 https://cdnjs.cloudflare.com/ajax/libs/socket.io/2.1.1/socket.io.dev.js
4. [Atom]更新public/index.html以在客户端引用和调用socket.io：
a. 在title下添加类似如下代码引用库： 
```
<script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/2.1.1/socket.io.dev.js"></script>
```
b. 在body中添加如下调用socket.io的外部脚本： 
```
<script src="chat.js"></script>
```
c. 创建外部脚本文件public/chat.js： 
```
var socket = io.connect('http://localhost:5000');
```
5. 刷新网页 http://localhost:5000/ 可以看到Web服务器控制台记录下每次不同的socket.id。

构建ChatApp主体
1. [Atom]更新index.html文件，在body部分添加如下容器定义：
```
 <div id="mario-chat"> 
  <div id="chat-window"> 
    <div id="output"></div> 
  </div> 
  <input type="text" id="handle" placeholder="Handle"> 
  <input type="text" id="message" placeholder="Message"> 
  <button id="send">Send</button> 
</div>
```
2. 刷新 http://localhost:5000/ 检查容器定义。
3. [Atom]更新 chat.js 添加如下socket.io的调用代码：
```
 var message = document.getElementById('message'), 
handle = document.getElementById('handle'), 
btn = document.getElementById('send'), 
output = document.getElementById('output');  

// 返回输入的message和handle数据到服务器 
btn.addEventListener('click', function(){ 
  socket.emit('chat', { message: message.value, handle: handle.value }); 
  message.value = ""; 
});

  // 更新来自服务器的chat相关数据到容器
output socket.on('chat', function(data){ 
  output.innerHTML += '<p><strong>' + data.handle + ': </strong>' + data.message + '</p>'; }
);
```
4. [Atom]更新index.js的io.on部分代码如下： 
```
// 获得来自客户端的数据，作为chat类型信息发送到客户端
 io.on('connection',function(socket){ 
  socket.on('chat', function(data){ 
    io.sockets.emit('chat', data); 
  });
 });
```
5. 分别打开两个网页到 http://localhost:5000/ ，验证基本聊天功能。

添加聊天状态内容
1. [Atom]更新index.html，在outpu容器下添加如下容器：
```
 <div id="feedback"></div>
```
2. [Atom]更新chat.js，在DOM变量定义output下追加feedback定义： 
```
feedback = document.getElementById('feedback') 
```
继续在btn事件监听下面定义对聊天内容输入的监听： 
```
// 对message对象监听输入操作，作为typing类型信息发送到服务器
 message.addEventListener('keypress', function(){ 
  socket.emit('typing', handle.value); }
);
```
3. [Atom]更新index.js的io.on内容，添加代码处理typing类型信息： 
```
// 这里使用广播方法处理接收到的typing类型信息 
socket.on('typing', function(data){ 
  socket.broadcast.emit('typing', data); 
});
```
4. [Atom]再次更新chat.js添加处理服务器返回的typing类型广播信息： 
```
socket.on('typing', function(data){ 
  feedback.innerHTML = '<p><em>' + data + 'is typing a message...</em></p>’; });
```
5. 测试可以发现，多方交替输入时后输入者的feedback容器没有清空。
6. [Atom]更新chat.js的处理chat数据的socket.on方法，添加在output前：
```
// 在输出服务器返回的chat信息前清空feedback广播 
feedback.innerHTML = “”;
```
