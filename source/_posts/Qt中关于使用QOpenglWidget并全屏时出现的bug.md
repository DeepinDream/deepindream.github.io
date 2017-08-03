---
layout: '[layout]'
title: Qt中关于使用QOpenglWidget并全屏时出现的bug
date: 2017-07-28 22:48:48
categories: Qt
tags: Qt
---

##### Tips：bug出现的软件环境：Win10，Qt5.7.2，msvc14编译器

#### 最近在用QOpenglWidget时，发现一旦showFullScreen()，就会出现无法打开QFileDialog的bug。仔细看并不是无法打开，而是打开了，但是无法渲染出来，就像透明了一样，然而鼠标移动过去是能发现弹窗是有打开的。

#### 这就很奇怪了，原本QDialog类弹窗是能自动显示在最前端的，出现这种问题的原因，很有可能是全屏时，QOpenglWidget本身是渲染于最前端的，导致与QFileDialog发生冲突。这T喵的就很尴尬的...

#### Google了一下，索引到QtBug论坛，发现这并不是个例，而是一个已知的bug，并且上面描述Qt5.4环境下就已经出现了，但是为什么到5.7还没修复啊摔(′д｀ )…彡…彡

```
Description
No popup, menu or tooltip widgets show up on a QOpenGLWidget/Window if it is in Fullscreen!
This is probably related to QTBUG-7556, however the suggested workaround no longer works!
Alt+Tab away from a Fullscreen QOpenGLWidget/Window will not bring other windows in front; will change input focus and mouse cursors, but all painting on screen will still be from QOpenGLWidget/Window.
Supplied is a sample, which actually implements the QTBUG-7556 workaround.
```

#### [Qt官方文档](http://doc.qt.io/qt-5/windows-issues.html)也有相应的描述


#### 既然发现了bug，自然是要解决的啦。方案如下：

-1. 修改源码O(∩_∩)O，没错就是得傲娇。
打开qt目录下的src/plugins/platforms/windows/qwindowswindow.cpp 

```c++
SetWindowPos(m_data.hwnd, HWND_TOP, r.left(), r.top(), r.width(), r.height(), swpf);
```

修改为：

``` c++
SetWindowPos(m_data.hwnd, HWND_TOP, r.left()-1, r.top()-1, r.width()+2, r.height()+2, swpf);
```

然而这方法我并没有试过。不确定可行。

-2. 比较简单的方法，尽管是治标不治本。

不使用showFullScreen()函数。
首先使用Qt::FramelessHint将边框隐藏，接着使用QApplication::desktop->screenGeometry()获取屏幕尺寸，再将当前窗口的尺寸相应修改就可以了。记得最后调用show()。