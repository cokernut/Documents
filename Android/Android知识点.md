---
title: Android知识点
date: 2017-01-11 09:58:28
tags: [Android]
categories: Android
description:
---
Android知识点
<!--more-->

### tools:context="activity name"作用

tools 相关的属性是提示给编辑器的，也就是用来辅助编辑器展示效果，在真机上这些属性是没有作用的。例如这里的 tools:context 就是将这个 layout 文件和后面的 Activity 进行关联，这样编辑器在展示布局效果的时候，就能针对 Activity 的一些属性进行有针对性的处理。

tools:context="activity name"这一句不会被打包进APK。只是ADT的Layout Editor在你当前的Layout文件里面设置对应的渲染上下文，说明你当前的Layout所在的渲染上下文是activity name对应的那个activity，如果这个activity在manifest文件中设置了Theme，那么ADT的Layout Editor会根据这个Theme来渲染你当前的Layout。就是说如果你设置的MainActivity设置了一个Theme.Light（其他的也可以），那么你在可视化布局管理器里面看到的背景阿控件阿什么的就应该是Theme.Light的样子。仅用于给你看所见即所得的效果而已。

### 横竖屏

在开发android的应用中，有时候需要限制横竖屏切换。只需要在AndroidManifest.xml文件中加入android:screenOrientation属性限制。

android:screenOrientation="landscape"是限制此页面横屏显示，
android:screenOrientation="portrait"是限制此页面数竖屏显示。

Android:screenOrientation设定该活动的方向，该值可以是任何一个下面的字符串：
+ "unspecified"
默认值，由系统选择显示方向，在不同的设备可能会有所不同。
+ "landscape"
橫向
+ "portrait"
纵向
+ "user"
用户当前首选的方向
+ "behind"
与在Activity栈中的Activity相同的方向
+ "sensor"
根据物理方向传感器确定方向，取决于用户手持的方向，当用户转动设备，它会跟随改变。
"nosensor"
不通过物理方向传感器确定方向，该传感器会被忽略，所以当用户转动设备，显示不会跟随改变，除了这个区别，系统选择根据“未指定”("unspecified")的方式选择显示方向。

如果要使Activity的View界面全屏，只需要将最上面的信号栏和Activity的Title栏隐藏掉即可，隐藏Title栏的代码：
requestWindowFeature(Window.FEATURE_NO_TITLE);
配置文件里代码：
android:theme="@android:style/Theme.NoTitleBar"
隐藏信号栏的代码：
getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN);
其它使用：
getWindow().setFlags(WindowManager.LayoutParams.TYPE_STATUS_BAR, WindowManager.LayoutParams.TYPE_STATUS_BAR);

### 软键盘

当在Android的layout设计里面如果输入框过多，则在输入弹出软键盘的时候，下面的输入框会有一部分被软件盘挡住，从而不能获取焦点输入。或者是有使用framentlayout悬浮在底部的button也会挡住输入框。

一、解决办法
方法一：
在你的activity中的oncreate中setContentView之前写上这个代码getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_PAN);

方法二：
在项目的AndroidManifest.xml文件中界面对应的<activity>里加入android:windowSoftInputMode="stateVisible|adjustResize"，这样会让屏幕整体上移。如果加上的 android:windowSoftInputMode="adjustPan"这样键盘就会覆盖屏幕。

方法三：
把顶级的layout替换成ScrollView，或者说在顶级的Layout上面再加一层ScrollView的封装。这样就会把软键盘和输入框一起滚动了，软键盘会一直处于底部。

总结：
我通常是方法二与方法三一起使用，肯定没有问题了，方法推荐不这么用。

不希望遮挡设置activity属性android:windowSoftInputMode="adjustPan"
希望动态调整高度android:windowSoftInputMode="adjustResize"

二、接下来看看，其他属性值的描述

1、"stateUnspecified"
软键盘的状态 (是否它是隐藏或可见 )没有被指定。系统将选择一个合适的状态或依赖于主题的设置。
这个是为了软件盘行为默认的设置。
2、"stateUnchanged"
软键盘被保持无论它上次是什么状态，是否可见或隐藏，当主窗口出现在前面时。
3、"stateHidden"
当用户选择该 Activity时，软键盘被隐藏——也就是，当用户确定导航到该 Activity时，而不是返回到它由于离开另一个 Activity。
4、"stateAlwaysHidden"
软键盘总是被隐藏的，当该 Activity主窗口获取焦点时。
5、"stateVisible"
软键盘是可见的，当那个是正常合适的时 (当用户导航到 Activity主窗口时 )。
6、"stateAlwaysVisible"
当用户选择这个 Activity时，软键盘是可见的——也就是，也就是，当用户确定导航到该 Activity时，而不是返回到它由于离开另一个 Activity。
7、"adjustUnspecified"
它不被指定是否该 Activity主 窗口调整大小以便留出软键盘的空间，或是否窗口上的内容得到屏幕上当前的焦点是可见的。系统将自动选择这些模式中一种主要依赖于是否窗口的内容有任何布局 视图能够滚动他们的内容。如果有这样的一个视图，这个窗口将调整大小，这样的假设可以使滚动窗口的内容在一个较小的区域中可见的。这个是主窗口默认的行为 设置。
8、"adjustResize"
该 Activity主窗口总是被调整屏幕的大小以便留出软键盘的空间
9、"adjustPan"
该 Activity主窗口并不调整屏幕的大小以便留出软键盘的空间。相反，当前窗口的内容将自动移动以便当前焦点从不被键盘覆盖和用户能总是看到输入内容的部分。这个通常是不期望比调整大小，因为用户可能关闭软键盘以便获得与被覆盖内容的交互操作。

### 手动显示和隐藏软键盘

1、方法一(如果输入法在窗口上已经显示，则隐藏，反之则显示)
[java] view plain copy print?
InputMethodManager imm = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);  
imm.toggleSoftInput(0, InputMethodManager.HIDE_NOT_ALWAYS);  

2、方法二（view为接受软键盘输入的视图，SHOW_FORCED表示强制显示）
[java] view plain copy print?
InputMethodManager imm = (InputMethodManager) getSystemService(Context.INPUT_METHOD_SERVICE);  
imm.showSoftInput(view,InputMethodManager.SHOW_FORCED);  
[java] view plain copy print?
imm.hideSoftInputFromWindow(view.getWindowToken(), 0); //强制隐藏键盘  

3、调用隐藏系统默认的输入法
[java] view plain copy print?
((InputMethodManager)getSystemService(Context.INPUT_METHOD_SERVICE)).hideSoftInputFromWindow(WidgetSearchActivity.this.getCurrentFocus().getWindowToken(), InputMethodManager.HIDE_NOT_ALWAYS);  (WidgetSearchActivity是当前的Activity)

4、获取输入法打开的状态
[java] view plain copy print?
InputMethodManager imm = (InputMethodManager)getSystemService(Context.INPUT_METHOD_SERVICE);  
boolean isOpen=imm.isActive();//isOpen若返回true，则表示输入法打开  