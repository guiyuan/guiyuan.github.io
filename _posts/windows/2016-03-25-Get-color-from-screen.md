---
layout: post
comments: true
title: Get color from screen
author: guiyuan
date: 2016-03-25 16:06:39
categories: Windows
keywords: BitBlt GetDIBits
description: 屏幕取色器

---

最近有个功能需要从屏幕取色，涉及到截获windows消息，以及取颜色的系统api。


#监测系统鼠标消息

首先是要截获系统消息，进入取色状态后，所有的鼠标消息都应该截获到，以便于确定鼠标位置，首先想到的就是钩子，注册一个鼠标钩子检测鼠标事件。

```
// 注册钩子
mouseHook = SetWindowsHookEx(WH_MOUSE_LL, MouseProc, GetModuleHandle(NULL), 0);

// 卸载钩子
UnhookWindowsHookEx(mouseHook);
```
鼠标钩子类型有两种，这里使用了WH\_MOUSE\_LL类型，还有一种WH\_MOUSE:

1.WH\_MOUSE\_LL 这个称为低级鼠标钩子，可以监测到整个系统的鼠标事件
2.WH\_MOUSE 这个只能监测到钩子所在模块的鼠标事件

MouseProc是自己处理鼠标消息的回调函数：

```
LRESULT CALLBACK MouseProc(int code, WPARAM wParam, LPARAM lParam)




刚开始用了API GetPixel,但是这个会很慢，消耗比较大，我在鼠标移动时一直需要调用。然后查了资料将屏幕拷贝到内存中，然后再去操作，这样就会很快了。

   首先是将屏幕内容拷贝到内存dc中：

```
	// 获取屏幕dc
```

然后通过GetDiBits调用得到内存中的数据信息到一个buffer，这个函数调用两次，第一次获取buffer的大小，第二次实际拷贝数据到buffer。

```
	if (GetDIBits(memDC, hbmScreen, 0, 0, NULL, &bmInfo, DIB_RGB_COLORS) == 0)



int b = buffer[((y * width) + x) * 4]
int g = buffer[((y * width) + x) * 4 + 1]
int r = buffer[((y * width) + x) * 4 + 2]
