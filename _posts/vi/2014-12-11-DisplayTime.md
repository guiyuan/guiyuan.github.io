---
layout: post
comments: true
title: 在vi中插入时间
author: guiyuan
date: 2014-12-11 11:43:15
category: vi
---

   
   刚开始想用vi来写markdown文件，在文件头中想插入当前的系统时间，在网上查了下可以用
	`:r !date`
来插入，不过格式不是我想要的，这种插入是形如`2014年12月11日 星期四 11时49分10秒 CST`这种格式。我想要的是`2014-12-11 11:43:15`这种格式的插入。

  在[CSDN](http://blog.csdn.net/linwhwylb/article/details/6284286)这篇文章中看到可以使用`strftime`来进行格式化输出，查了下`strftime`的用法，在.vimrc中添加如下两行就可以达到我的目标了：
 
 	:nnoremap<F5> "=strftime("%F %T")<CR>gP
 	:inoremap<F5> <C-R>=strftime("%F %T")<CR>
 	
 第一行是在正常模式下插入，第二行是在编辑模式下进行插入。
 
 下面列一些`strftime`的格式化命令说明（摘自[百度百科](http://baike.baidu.com/link?url=C-ge9XweA4tpnGvpPNFZA5cok5wy_epKdGXX8Fj6hFLXvUwEL_TtWfZLWzuXzQWzifSEhq_CChpO8PJ-LLk7oK)）：
 	
 	%a 星期几的简写
	%A 星期几的全称
	%b 月份的简写
	%B 月份的全称
	%c 标准的日期的时间串
	%C 年份的前两位数字
	%d 十进制表示的每月的第几天
	%D 月/天/年
	%e 在两字符域中，十进制表示的每月的第几天
	%F 年-月-日
	%g 年份的后两位数字，使用基于周的年
	%G 年份，使用基于周的年
	%h 简写的月份名
	%H 24小时制的小时
	%I 12小时制的小时
	%j 十进制表示的每年的第几天
	%m 十进制表示的月份
	%M 十时制表示的分钟数
	%n 新行符
	%p 本地的AM或PM的等价显示
	%r 12小时的时间
	%R 显示小时和分钟：hh:mm
	%S 十进制的秒数
	%t 水平制表符
	%T 显示时分秒：hh:mm:ss
	%u 每周的第几天，星期一为第一天 （值从1到7，星期一为1）
	%U 第年的第几周，把星期日作为第一天（值从0到53）
	%V 每年的第几周，使用基于周的年
	%w 十进制表示的星期几（值从0到6，星期天为0）
	%W 每年的第几周，把星期一做为第一天（值从0到53）
	%x 标准的日期串
	%X 标准的时间串
	%y 不带世纪的十进制年份（值从0到99）
	%Y 带世纪部分的十制年份
	%z，%Z 时区名称，如果不能得到时区名称则返回空字符。
	%% 百分号
