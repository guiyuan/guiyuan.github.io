---
layout: post
comments: true
title: Authorization in nginx-rtmp-module
author: guiyuan
date: 2015-09-11 15:21:38
categories: Nginx
keywords: Nginx, rtmp, Authorization
description: 

---
任何人都可以任意访问服务器肯定是不行的，因此我们需要做出一些限制，只有我们期望的用户才可以。要控制访问流媒体服务器的权限可以通过nginx-rtmp-module中的on_play,on_publish系列的命令来进行，可以参见[wiki](thub.com/arut/nginx-rtmp-module/wiki/Directives).

<pre><code>application myapp {
	live on;
	
	on_play http://localhost/onPlay.php;
	on_publish http://localhost/onPublish.php
}</code></pre>

当用户在上传rtmp流时，服务器在接收到客户端的连接后，首先执行on_publish指定的回调，可以在串流码中添加所需要的参数，然后在回调中通过get方法得到，进而进行所需要的验证处理，当返回2xx的http返回码时表示通过验证，返回3xx时表示要重定向，重定向的地址在header的Location中，返回其他代码服务器回断开连接。

<pre><code>传递参数：
	URL:rtmp://server/myapp
	streamkey: mystream?user=username&pwd=password
</code></pre>
在回调中就可以通过get方法得到user和pwd的值，同时还可以得到一系列的服务器固定给的信息，如下所示：
<pre><code>{"app":"myapp","flashver":"MAC 18,0,0,232","swfurl":"http:\/\/localhost\/rtmp-publisher\/RtmpPlayer.swf\/[    [DYNAMIC]]\/5","tcurl":"rtmp:\/\/127.0.0.1:1935\/myapp","pageurl":"http:\/\/localhost\/rtmp-publisher\/pla    yer.html","addr":"127.0.0.1","clientid":"5","call":"play_done","name":"mystream"}
</code></pre>

这些信息可以根据需求来决定是否适用。
   
