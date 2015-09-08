---
layout: post
comment: true
title: nginx-rtmp-module
author: guiyuan
date: 2015-09-08 23:24:15 
category: Nginx
keywords: Nginx, rtmp
description: Install nginx-rtmp-module

---

最近工作中要接触到直播相关的内容，研究了一下nginx-rtmp-module这个nginx的模块，他可以实现rtmp以及hls的流媒体服务。

记录一下安装过程:

+	首先要安装openssl和pcre，去对应的官网下载，nginx需要这个
+	下载nginx的最新[版本](http://nginx.org/en/download.html)
+	下载nginx-rtmp-module的[源码](https://github.com/arut/nginx-rtmp-module)
+	进入nginx源码目录，执行下列命令
<pre><code>
	./configure --add-module=/path/to/nginx-rtmp-module --with-pcre=/path/to/pcre 
	make
	make install
</code></pre>

如果遇到下面这种错误
<pre>
ngx_crypt.c:117:13: error: 'MD5_Update' is deprecated: first deprecated in OS X 10.7 [-Werror,-Wdeprecated-declarations]
</pre>

则在./configure 后面添加上 --with-cc-opt="-Wno-deprecated-declarations"即可。

安装完成后需要配置nginx.conf,在nginx-rtmp-module源码目录下test目录里有例子，具体可以参考[wiki](https://github.com/arut/nginx-rtmp-module/wiki/Directives)

配置完成后启动nginx，没什么错误就ok了。

刚开始启动后不知道怎么停掉，查了资料后记录一下：

先执行：
<pre>ps -ef | grep nginx
</pre>
查看进程号，要看master的，然后执行
<pre>sudo kill -QUIT 进程号</pre>


   

