# 本仓库为使用peerJS实现点对点聊天的例子。
## PeerJS介绍
>peerJS 封装了浏览器的 [WebRTC](https://github.com/webrtc) 实现，以提供完整、可配置且易于使用的点对点连接 API。只需要配置一个 ID，就可以创建到远程对等点的 P2P 数据或媒体流连接。
简单来说就是PeerJS实现了让浏览器之前**相互**通信更加便捷。

本文将演示如何实现用户A进入网页后生成一个链接，把链接发送给用户B，然后A与B就能相互聊天。

## 先介绍几个必备的方法。

1.引入PeerJS库。（还可以直接下载[源文件](https://unpkg.com/peerjs@1.3.1/dist/peerjs.min.js)）
```javascript
<script src="https://unpkg.com/peerjs@1.3.2/dist/peerjs.min.js"></script>
```

2.创建一个[Peer对象](https://peerjs.com/docs/#peer)

```bash
var peer = new Peer();
这里可以传参数,第一个参数是id，第二个参数是由选项组成的对象。详情见文档。
```
3.创建完成的回调

```bash
peer.on('open', function(id) {
	console.log('My peer ID is: ' + id); //输出自己的id，后面别人连接自己要用。
  });
```
4.建立连接

```bash
var conn = peer.connect('id'); //id表示连接对象的id
```
5.监听被连接

```bash
peer.on('connection', function(conn) {
	console.log("被连接",conn);
    receiveId = conn.peer; //发起连接者的id
});
```
6.建立连接后的回调，这里的conn.on('data'）表示监听消息。

```bash
conn.on('open', function() {
	//监听接收消息
	conn.on('data', function(data) { 
	  console.log('Received', data);
	});
	//发送消息
	conn.send('Hello!'); 
  });
```

## 下面有一个简单的例子：

> self.html 表示**聊天发起端**（startId）
target.html表示**聊天接收端**（receiveId）

流程：
1.首先是**聊天发起端**创建一个peer对象，然后监听被连接，被连接后监听是否收到消息。
2.**聊天接收端**通过带有**发起端id**的url进入，进入后从url解析出**发起端id**，再和**发起端**建立连接，建立连接成功后直接发一个信息。
从下图可看出，聊天发送端接收到了信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/f6bb4cbcdb4e42859296b724c8623130.png)
**代码如下：**

```bash
								self.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>聊天发起端</title>
  <script src="https://unpkg.com/peerjs@1.3.2/dist/peerjs.min.js"></script>
  <!-- <script src="/js/peerjs.min.js"></script> -->
  <script>
    var peer = null; 
    var startId = null; 
    var receiveId = null;
    var conn = null;
    //页面加载后 创建Peer对象 生成另外一段的链接  并监听连接和消息
    window.onload = function () {
      peer = new Peer(); //第一个参数为id 第二个参数为选项 ('',{"debug":3})
      console.log(peer)

      peer.on('open', function(id) {
        startId = id;
        console.log('My peer ID is: ' + id);
        console.log(`接收端链接${location.href.replace("self","target")}?startId=${startId}`)
      });

      peer.on('connection', function(conn) {
        console.log("被连接",conn);
        receiveId = conn.peer;
        conn.on('data', (data) => {
            console.log("收到消息",data);
        });
      });
    }
  </script>
</head>
<body> 
</body>
</html>
```

```bash
								target.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>聊天接收端</title>
  <script src="https://unpkg.com/peerjs@1.3.2/dist/peerjs.min.js"></script>
  <!-- <script src="/js/peerjs.min.js"></script> -->
  <script>
    var peer = null; 
    var startId = null; 
    var receiveId = null;
    var conn = null;

    //获取url后面的参数
    getRequest = function() { 
        var url = location.search; 
        var theRequest = new Object(); 
        if (url.indexOf("?") != -1) { 
            let str = url.substr(1); 
            let strs = str.split("&"); 
            for(let i = 0; i < strs.length; i++) { 
                theRequest[strs[i].split("=")[0]]=unescape(strs[i].split("=")[1]); 
            } 
        } 
        return theRequest; 
    },

    //页面加载后 创建Peer对象 主动建立连接 并发送一个”我来了“
    window.onload = function () {
      //获取参数 获取发起对话者的id 用户建立连接
      param = getRequest();
      startId = param.startId;
      peer = new Peer(); //第一个参数为id 第二个参数为选项 ('',{"debug":3})
      console.log(peer)
      peer.on('open', function(id) {
        receiveId = id;
        console.log('My peer ID is: ' + id);
        //这里先向start建立连接，让start知道这个端的id（reveivedId）
        conn = peer.connect(startId);
        conn.on('open', () => {
              //直接发送消息
              console.log("发送消息")
              conn.send("我来了");
        });
      });
      peer.on('connection', function(conn) {
        console.log("被连接",conn);
        conn.on('data', (data) => {
            console.log("收到消息",data);
        });
      });

    }

  </script>
</head>
<body>
</body>
</html>
```

## 下面再贴出一个完整的点对点聊天的例子
下效果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/f226dcfd223047848f33ae311167a20b.png)
**代码同样只有两个html文件**

```bash
								self.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>聊天发起端</title>
  <!-- <script src="https://unpkg.com/peerjs@1.3.2/dist/peerjs.min.js"></script> -->
  <script src="/js/peerjs.min.js"></script>
  <script>
    var peer = null; 
    var startId = null; 
    var receiveId = null;
    var conn = null;

    //输入框回车处理
    function keyup_submit(e){ 
      var evt = window.event || e; 
      if (evt.keyCode == 13){
          sendMessage()
        }
    }
    //发送消息的函数
    sendMessage = function () {
      receiveId = receiveId?receiveId:localStorage.getItem("receiveId")
      if(!receiveId){
        alert("你没有聊天的对象，请把连接发给对象");
        return
      }
      // console.log(document.getElementById("messageText").value)
      let message = document.getElementById("messageText").value;
      //消息体
      if (!conn) {
          //创建到对方的连接
          console.log("发起连接",receiveId)
          conn = peer.connect(receiveId);
          conn.on('open', () => {
              //首次发送消息
              conn.send(message);
              //把信息添加到页面
              document.getElementById("chatContext").innerHTML += `<div class="right"><span>${message}</span></div>`
              document.getElementById("messageText").value = null
            });
      }

      //发送消息
      if (conn.open) {
          conn.send(message);
          //把信息添加到页面
          document.getElementById("chatContext").innerHTML += `<div class="right"><span>${message}</span></div>`
          document.getElementById("messageText").value = null
        }
      //移动到消息底部
      container = document.getElementById("chatContext")
      container.scrollTop = container.scrollHeight;
    }

    //页面加载后 创建Peer对象 生成另外一段的链接  并监听连接和消息
    window.onload = function () {
      //加载聊天记录
      document.getElementById("chatContext").innerHTML += JSON.parse(localStorage.getItem("chatContext"))
      //每次使用同一个id
      if(localStorage.getItem("startId")=="null"){
        startId = new Date().getTime();
        localStorage.setItem("startId",startId)
      }else{
        startId = localStorage.getItem("startId")
      }
      // alert(startId)
      peer = new Peer(startId); //第一个参数为id 第二个参数为选项 ('',{"debug":3})
      console.log(peer)
      //Every Peer object is assigned a random, unique ID when it's created.
      peer.on('open', function(id) {
        startId = id;
        console.log('My peer ID is: ' + id);
        // console.log(`http://${location.host}/target.html?startId=${startId}`)
        // console.log(location.href.replace("self","target"))
        document.getElementById("targetUrl").innerHTML =`<a target="_blank" href="${location.href.replace("self","target")}?startId=${startId}">${location.href.replace("self","target")}?startId=${startId}</a>` 
      });

      peer.on('connection', function(conn) {
        console.log("被连接",conn);
        receiveId = conn.peer;
        localStorage.setItem("receiveId",receiveId); //防止刷新不见了
        conn.on('data', (data) => {
            console.log("收到消息",data);
            //把信息添加到页面
            document.getElementById("chatContext").innerHTML += `<div class="left"><span>${data}</span></div>`
        });
      });
    
    }

    //页面刷新前保存聊天记录
    window.addEventListener("beforeunload", function (e) {
      localStorage.setItem("chatContext",JSON.stringify(document.getElementById("chatContext").innerHTML))
    });
  </script>
</head>
<body>
  <div class="container">
    <h2 align="center">基于<a target="_blank" href="https://peerjs.com">PeerJS</a>的点对点聊天Demo</h2>
    把下方链接发送给你对象即可开始聊天啦
    <div>对象链接:<span id="targetUrl">生成中......</span></div>
    <div id="chatContext">
    </div>
    <div class="bottom">
      <input style="width: 80%;height: 30px;" type="text" onkeydown="keyup_submit(event)" name="" id="messageText">
      <button style="height: 36px;" onclick="sendMessage()">发 送</button>
    </div>
  </div>
  
</body>
<style>
  .container {
    position: relative;
    max-width: 360px;
    height: 80vh;
    margin: 0 auto;
    border: 3px solid #ccc;
    padding: 10px;
    background-color: #F5F5F5;
  }
  #chatContext {
    border-top: 2px solid #ccc;
    padding: 10px 0;
    height: 50vh;
    overflow-y: scroll;
  }
  #chatContext:after{
    content:"";
    display:block;
    visibility:hidden;
    clear:both;
  }
  .left {
    text-align: left;
    margin: 15px;
  }
  .left span {
    background-color: #FFFFFF;
    padding: 4px 8px;
  }
  .right {
    text-align: right;
    margin: 15px;
  }
  .right span{
    background-color: #9EEA6A;
    padding: 4px 8px;
  }
  .bottom {
    width: 100%;
    position: absolute;
    bottom: 20px;
  }
</style>
</html>
```

```bash
								target.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>聊天接收端</title>
  <!-- <script src="https://unpkg.com/peerjs@1.3.2/dist/peerjs.min.js"></script> -->
  <script src="/js/peerjs.min.js"></script>
  <script>
    var peer = null; 
    var startId = null; 
    var receiveId = null;
    var conn = null;

    //输入框回车处理
    function keyup_submit(e){ 
      var evt = window.event || e; 
      if (evt.keyCode == 13){
          sendMessage()
        }
    }
    //发送消息的函数
    sendMessage = function () {
        console.log("sendMessage")
        let message = document.getElementById("messageText").value;
        //消息体
        // var message = { "from": receiveId, "to": startId, "body": document.getElementById("messageText").value };
        if (!conn) {
            //创建到对方的连接
            conn = peer.connect(startId);
            conn.on('open', () => {
                //首次发送消息
                conn.send(message);
                //把信息添加到页面
                document.getElementById("chatContext").innerHTML += `<div class="right"><span>${message}</span></div>`
                document.getElementById("messageText").value = null
            });
        }

        //发送消息
        if (conn.open) {
           conn.send(message);
           //把信息添加到页面
           document.getElementById("chatContext").innerHTML += `<div class="right"><span>${message}</span></div>`
           document.getElementById("messageText").value = null
        }

        //移动到消息底部
        container = document.getElementById("chatContext")
        container.scrollTop = container.scrollHeight;
    }

    //获取url后面的参数
    getRequest = function() { 
        var url = location.search; 
        var theRequest = new Object(); 
        if (url.indexOf("?") != -1) { 
            let str = url.substr(1); 
            let strs = str.split("&"); 
            for(let i = 0; i < strs.length; i++) { 
                theRequest[strs[i].split("=")[0]]=unescape(strs[i].split("=")[1]); 
            } 
        } 
        return theRequest; 
    },

    //页面加载后 创建Peer对象 主动建立连接 并发送一个”我来了“
    window.onload = function () {
      //加载聊天记录
      document.getElementById("chatContext").innerHTML += JSON.parse(localStorage.getItem("chatContext"))
      //获取参数
      param = getRequest();
      startId = param.startId;
      //每次使用同一个id
      if(!localStorage.getItem("receiveId")){
        receiveId = new Date().getTime();
        localStorage.setItem("receiveId",receiveId)
      }else{
        receiveId = localStorage.getItem("receiveId")
      }
      peer = new Peer(receiveId); //第一个参数为id 第二个参数为选项 ('',{"debug":3})
      console.log(peer)
      peer.on('open', function(id) {
        receiveId = id;
        console.log('My peer ID is: ' + id);
        //这里先想start建立连接，让start知道这个端的id（reveivedId）
        conn = peer.connect(startId);
        conn.on('open', () => {
              //首次发送消息
              console.log("发送消息")
              conn.send("我来了");
              document.getElementById("chatContext").innerHTML += `<div class="right"><span>我来了</span></div>`
        });
      });
      peer.on('connection', function(conn) {
        console.log("被连接",conn);
        conn.on('data', (data) => {
            console.log("收到消息",data);
            document.getElementById("chatContext").innerHTML += `<div class="left"><span>${data}</span></div>`
        });
      });

    }

    //页面刷新前保存聊天记录
    window.addEventListener("beforeunload", function (e) {
      localStorage.setItem("chatContext",JSON.stringify(document.getElementById("chatContext").innerHTML))
    });
  </script>
</head>
<body>
  <div class="container">
    <h2 align="center">基于<a target="_blank" href="https://peerjs.com">PeerJS</a>的点对点聊天Demo</h2>
    <div id="chatContext">
    </div>
    <div class="bottom">
      <input style="width: 80%;height: 30px;" type="text" onkeydown="keyup_submit(event)" name="" id="messageText">
      <button style="height: 36px;" onclick="sendMessage()">发 送</button>
    </div>
  </div>
  
  
</body>
<style>
  .container {
    position: relative;
    max-width: 360px;
    height: 80vh;
    margin: 0 auto;
    border: 3px solid #ccc;
    padding: 10px;
    background-color: #F5F5F5;
  }
  #chatContext {
    border-top: 2px solid #ccc;
    padding: 10px 0;
    height: 60vh;
    overflow-y: scroll;
  }
  #chatContext:after{
    content:"";
    display:block;
    visibility:hidden;
    clear:both;
  }
  .left {
    text-align: left;
    margin: 15px;
  }
  .left span {
    background-color: #FFFFFF;
    padding: 4px 8px;
  }
  .right {
    text-align: right;
    margin: 15px;
  }
  .right span{
    background-color: #9EEA6A;
    padding: 4px 8px;
  }
  .bottom {
    width: 100%;
    position: absolute;
    bottom: 20px;
  }
</style>
</html>
```
源码地址：[https://github.com/QiuYang01/Chat-Base-On-PeerJS/tree/master](https://github.com/QiuYang01/Chat-Base-On-PeerJS/tree/master)