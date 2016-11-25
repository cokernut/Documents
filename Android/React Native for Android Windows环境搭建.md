Title: React Native for Android Windows环境搭建
Date: 2016-11-23
Modified: 2016-11-23
Tags: Android,React Native
Slug: Android_React_Native_Windows_Environment
Authors: Cokernut
Summary: React Native for Android Windows环境搭建

## 安装JDK

从[Java官网](http://www.oracle.com/technetwork/cn/java/javase/downloads/jdk8-downloads-2133151-zhs.html)下载JDK并安装

设置环境变量;JAVA_HOME环境变量设置为你jdk所在目录，将JDK的bin目录加入系统PATH环境变量

## 安装Android SDK

安装Android SDK，把ANDROID_HOME环境变量设置为你sdk所在目录，将SDK的platform-tools和tools目录加入系统PATH环境变量  
这里你可以安装Android Studio, Android Studio包含了运行和测试React Native应用所需的Android SDK和模拟器。React Native目前需要Android Studio2.0或更高版本，
Android Studio需要Java Development Kit [JDK] 1.8或更高版本。

## 安装C++环境

下载并安装Visual Studio。也可选择Windows SDK、cygwin或mingw等其他C++环境。编译node.js的C++模块时需要用到

## 安装[git](https://git-scm.com/downloads)客户端

init那一步需要用到这个东西  

## 安装Python

从官网下载并安装[python](https://www.python.org/downloads/) 2.7.x（3.x版本目前不支持）

## 安装Nodejs

请到[官网](https://nodejs.org/en/)下载Windows版本的Node.js
我选的是最新版，React Native的要求是4.1以上都可以支持，注意，目前已知Node 7.1版本在windows上无法正常工作，请注意避开这个版本！  
建议设置npm镜像以加速后面的过程（或使用科学上网工具）
> npm config set registry https://registry.npm.taobao.org --global  
> npm config set disturl https://npm.taobao.org/dist --global  
## 安装react-native-cli（react-native命令行工具）

Windows命令行运行：
> npm install -g react-native-cli   

注意：有的教程命令是：npm install -g yarn react-native-cli就是多装了一个yarn模块，
Yarn是Facebook提供的替代npm的工具，可以加速node模块的下载，它是在我们使用npm命令下载模块的时候
会间接调用yarn命令下载，但是国内因为访问不了Facebook(梯子另说)，所以用yarn下载的结果要么是下载不了，
要么是比npm还慢。所以你发现init项目的时候很慢或者是进度不知道动，那么你看一下命令行是不是因为有当你用npm的时候
下面有说明调用yarn了，还有个版本号。这种情况下我们应该把yarn进行删除，删除方法：默认是在命令行下转到C:\Users\UserName\AppData\Roaming\npm
目录下运行:
> npm uninstall yarn  

这样就可以把yarn模块进行删除，我们使用npm命令时就不会调用yarn了。
React Native的命令行工具用于执行创建、初始化、更新项目、运行打包服务（packager）等任务。
（进度慢的同学可以尝试使用Vpn或者是其他加速方法-_-）  

## 初始化项目

进入你的工作目录，Windows命令行运行：

> react-native init Project  

耐心等待。。。。。。此过程有点慢！

## 启动

进入项目目录（上面的Project目录）启动React Native Server
> react-native start  

可以用浏览器访问http://localhost:8081/index.android.bundle?platform=android看看是否可以看到打包后的脚本。第一次访问通常需要十几秒，并且在packager的命令行可以看到进度条

## 运行模拟器

启动Android模拟器或者是使用自己连接电脑  

## 安卓运行

在项目目录下运行
> react-native run-android  

首次运行需要等待数分钟并从网上下载gradle依赖
### 前提条件：USB调试

你需要开启USB调试才能在你的设备上安装你的APP。首先，确定你已经打开设备的USB调试开关

确保你的设备已经成功连接。可以输入adb devices来查看:
```
$ adb devices
List of devices attached
emulator-5554 offline   # Google模拟器
14ed2fcc device         # 真实设备
```
在右边那列看到device说明你的设备已经被正确连接了。  
<font color="#ff6559">注意，你只应当连接仅仅一个设备，如果你连接了多个设备（包含模拟器在内），
后续的一些操作可能会失败。拔掉不需要的设备，或者关掉模拟器，确保adb devices的输出只有一个是连接状态。</font>

现在你可以运行react-native run-android来在设备上安装并启动应用了。

注意：在真机上运行时可能会遇到白屏的情况，请找到并开启<font color="#ff6559">悬浮窗权限。</font>

## 从设备上访问开发服务器。

在启用开发服务器的情况下，你可以快速的迭代修改应用，然后在设备上查看结果。按照下面描述的任意一种方法来使你的运行在电脑上的开发服务器可以从设备上访问到。

注意：现在很多安卓设备已经没有了硬件"Menu"按键，这是我们用来调出开发者菜单的。在这种情况下你可以通过摇晃设备来打开开发者菜单(重新加载、调试，等等……)  

### (Android 5.0及以上)使用adb reverse命令

<font color="#ff6559">注意，这个选项只能在5.0以上版本(API 21+)的安卓设备上使用。</font>

首先把你的设备通过USB数据线连接到电脑上，并开启USB调试（设置-->开发者选项-->USB调试）。

> 运行<font color="#c7254e">adb reverse tcp:8081 tcp:8081</font>  
> 不需要更多配置，你就可以使用Reload JS和其它的开发选项了。  

### (Android 5.0以下)通过Wi-Fi连接你的本地开发服务器  
1. 首先确保你的电脑和手机设备在同一个Wi-Fi环境下。
2. 在设备上运行你的React Native应用。和打开其它App一样操作。
3. 你应该会看到一个“红屏”错误提示。这是正常的，下面的步骤会解决这个报错。
4. 摇晃设备，或者运行adb shell input keyevent 82，可以打开开发者菜单。
5. 点击进入Dev Settings。
6. 点击Debug server host for device。
7. 输入你电脑的IP地址和端口号（譬如10.0.1.1:8081）。在Mac上，你可以在系统设置/网络里找查询你的IP地址。在Windows上，打开命令提示符并输入ipconfig来查询你的IP地址。在Linux上你可以在终端中输入ifconfig来查询你的IP地址。
8. 回到开发者菜单然后选择Reload JS。  

## 异常 ：
![异常](images/Android/Android_React_Native_Windows_Environment/1.png "异常")  
 
异常 ReferenceError:Can't find variable:_fbBatchedBridge或者Unable to download JS bundle解决方法
看到这个，使劲摇晃手机或者直接点击菜单按钮，在出来的菜单里选择“Dev Settings”，然后点击最下面
的“Debug server host & port for device“，然后填入你电脑的ip:8081，你的手机和你的电脑
必须在同一个局域网内。设置完成以后再重启应用，你就可以看到React Native的欢迎界面了，
就是index.android.js页面的内容  
![效果](images/Android/Android_React_Native_Windows_Environment/2.png "效果")  

## 遇到的坑：

### 小米MIUI系统可能会遇到下面的问题：
react-native run-android的时候出现Failed to establish session问题：
解决方案：小米手机设置里-------开发者选项---------启用MIUI优化 关闭  
![异常](images/Android/Android_React_Native_Windows_Environment/3.png "异常")    
![异常](images/Android/Android_React_Native_Windows_Environment/4.png "异常")  
### 注意：以下是使用Genymotion时的情况，上面的解决方法无法解决
可以看到genymotion模拟器使用的网络连接方式是Host-Only方式，此情况下
Debug server host & port for device中填入的应该是VirtualBox Host-Only Network
的IP:8081（192.168.56.1:8081）,之后再重启应用或者是返回一下再点击菜单选择Reload JS,
你就可以看到React Native的欢迎界面了  
![异常](images/Android/Android_React_Native_Windows_Environment/5.png "异常")   
![异常](images/Android/Android_React_Native_Windows_Environment/6.png "异常")  
![异常](images/Android/Android_React_Native_Windows_Environment/10.png "异常") 

### 错误Could not connect to development server：
如果确定IP地址设置正确了还提示：Could not connect to development server，那么你要确定你的React Native Server启动了，没启动的话启动React Native Server，在项目目录下运行：
> react-native start  

![异常](images/Android/Android_React_Native_Windows_Environment/7.png "异常")   

## 安卓调试

打开Chrome，访问 http://localhost:8081/debugger-ui，应当能看到一个页面。按F12打开开发者菜单选择Sources勾选Pause On Caught Exceptions（如图）

在模拟器或真机菜单中选择Debug JS，即可开始调试  
![效果](images/Android/Android_React_Native_Windows_Environment/8.png "效果")  
![效果](images/Android/Android_React_Native_Windows_Environment/9.png "效果")  



