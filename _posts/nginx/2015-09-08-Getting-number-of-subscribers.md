---
layout: post
comments: true
title: Getting number of subscribers
author: guiyuan
date: 2015-09-08 23:24:51
category: Nginx
keywords: Nginx, rtmp
description: Getting number of subscribers

---

想要在nginx－rtmp中统计某个频道的观看数量的又一个简单的方法。

1. 需要在编译nginx的时候添加--with-http_xslt_module选项，即：
	<pre>./configure --with-http_xslt_module ...other </pre>
	
2. 创建一个简单的xsl文件nclient.xsl用语提取频道的当前观看数量，内容如下：
	<pre><code><xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
	<xsl:output method="html" />
	<xsl:param name="app" />
	<xsl:param name="name" />
	<xsl:template match="/">
		<xsl:value-of select="count(//application[name=%app]/live/stream[name=$name]/client[not(publishing) and flashver])" />
	</xsl:template>
	</xsl:stylesheet>
	</code></pre>

3. 修改nginx.conf配置文件

	设置静态页面配置
	<pre>location /stat {
		rtmp all;
		allow 127.0.0.1;
	}</pre>	
	添加一个页面请求返回统计的数量
	<pre>location /nclients {
		proxy_pass http://127.0.0.1/stat;
		xslt_stylesheet /path/to/nclient.xsl app='$arg_app' name='$arg_name';
		add_header Refresh "3; $request_uri";
	}</pre>
	xslt_stylesheet后面就是第二步创建的nclient.xsl文件的路径。

4. 重启nginx，然后通过下面的地址就可以得到频道观看数量
	<pre>http://server.com/nclients?app=myapp&name=mystream</pre>
	如果是在浏览器或者ifream中打开的话，统计结果会每3秒刷新一次。

	
