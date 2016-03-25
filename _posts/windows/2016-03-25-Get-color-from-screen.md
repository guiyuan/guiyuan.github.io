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
鼠标钩子类型有两种，这里使用了WH_MOUSE_LL类型，还有一种WH_MOUSE:
1.WH_MOUSE_LL 这个称为低级鼠标钩子，可以监测到整个系统的鼠标事件
2.WH_MOUSE 这个只能监测到钩子所在模块的鼠标事件

MouseProc是自己处理鼠标消息的回调函数：

```
LRESULT CALLBACK MouseProc(int code, WPARAM wParam, LPARAM lParam){	if (code < 0)	{		return CallNextHookEx(App()->getMouseHook(), code, wParam, lParam);	}	POINT p = ((MOUSEHOOKSTRUCT*)lParam)->pt;	switch (wParam)	{	case WM_LBUTTONUP:		// 鼠标左键抬起		break;	case WM_MOUSEMOVE:	{		// 鼠标移动		return CallNextHookEx(App()->getMouseHook(), code, wParam, lParam);	}	break;	case WM_RBUTTONUP:		// 鼠标右键抬起		break;	}	return 1;}
```
code小于0必须调用CallNextHookEx并返回。返回非0值就会截断消息，消息就不会传递下去。这里我在鼠标按下消息返回非0值，就会导致不会触发其他的操作，直到我去掉这个钩子，这样就便于鼠标移动取色。
# 取色

刚开始用了API GetPixel,但是这个会很慢，消耗比较大，我在鼠标移动时一直需要调用。然后查了资料将屏幕拷贝到内存中，然后再去操作，这样就会很快了。

   首先是将屏幕内容拷贝到内存dc中：

```
	// 获取屏幕dc	HDC deskDC = GetDC(NULL);	HDC memDC = CreateCompatibleDC(deskDC);	HBITMAP hbmScreen = NULL;	int w = GetDeviceCaps(deskDC, HORZRES);	mScreenX = w;	int h = GetDeviceCaps(deskDC, VERTRES);	mScreenY = h;	hbmScreen = CreateCompatibleBitmap(deskDC, w, h);	SelectObject(memDC, hbmScreen);	if (!BitBlt(memDC, 0, 0, w, h, deskDC, 0, 0, SRCCOPY))	{		ReleaseDC(0, deskDC);		DeleteDC(memDC);		DeleteObject(hbmScreen);		return;	}
```

然后通过GetDiBits调用得到内存中的数据信息到一个buffer，这个函数调用两次，第一次获取buffer的大小，第二次实际拷贝数据到buffer。

```
	if (GetDIBits(memDC, hbmScreen, 0, 0, NULL, &bmInfo, DIB_RGB_COLORS) == 0)	{		// ERROR handling		ReleaseDC(0, deskDC);		DeleteDC(memDC);		DeleteObject(hbmScreen);		return;	}	// create the pixel buffer	BYTE* buffer = new BYTE[bmInfo.bmiHeader.biSizeImage];	bmInfo.bmiHeader.biBitCount = 32;	bmInfo.bmiHeader.biCompression = BI_RGB;	bmInfo.bmiHeader.biHeight = abs(bmInfo.bmiHeader.biHeight);	if (0 == GetDIBits(memDC, hbmScreen, 0, bmInfo.bmiHeader.biHeight, buffer, &bmInfo, DIB_RGB_COLORS))	{		// error handling		ReleaseDC(0, deskDC);		DeleteDC(memDC);		DeleteObject(hbmScreen);		delete buffer;		buffer = NULL;		return;	}
```
这样buffer中存放的就是所有的颜色信息了。然后根据鼠标坐标点来取颜色值。颜色值的顺序是blue，green，red，还有一个字节的无用值。
```
int b = buffer[((y * width) + x) * 4]
int g = buffer[((y * width) + x) * 4 + 1]
int r = buffer[((y * width) + x) * 4 + 2]```
需要注意的一点是buffer是从屏幕的左下角开始的，而鼠标钩子截取的鼠标坐标是从屏幕左上角为坐标原点，因此y坐标需要转化一下。