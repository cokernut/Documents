---
title: React Native嵌入Android原生项目中
date: 2016-11-25
tags: [Android, React Native]
categories: React Native
description:
---
React Native嵌入Android原生项目中
<!--more-->

## 开发环境准备
首先你要搭建好React Native for Android开发环境， 没有搭建好的可以参考：[React Native for Android Windows环境搭建](http://cokernut.top/2016/11/23/Android/React%20Native%20for%20Android%20Windows%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/)  

## 用Android Studio新建Android原生项目
我创建了一个名叫ReactNativeDemo的原生项目。

## 把React Native集成到原生项目当中

### 利用Windows命令行在项目根目录(ReactNativeDemo文件夹)下执行下面三行命令：
> npm init  
> npm install --save react react-native  
> curl -o .flowconfig https://raw.githubusercontent.com/facebook/react-native/master/.flowconfig  

这将在项目根目录(ReactNativeDemo文件夹)创建node_modules文件夹(模块)，并添加React Native依赖。

## 针对上面三条命令的解释
### npm init
![图片](/images/ReactNative/React_Native_to_Native/1.png "图片")  
![图片](/images/ReactNative/React_Native_to_Native/2.png "图片")  
注意:  
name的填写由图可知填默认的是不行的，它的要求是不能有大写字母并且不能以数字开头；  
entry point的填写入口文件名称，默认的是index.js，我们建立的入口文件是index.android.js，所以填写index.android.js。只要填写的名称与自己定义的入口文件名称一致就行。  
其他的项根据自己需求填写即可。  

这个步骤会在项目的根目录产生一个名称为package.json的文件，我们还需要修改我们的package.json文件：
在"scripts"节点下添加"start": "node node_modules/react-native/local-cli/cli.js start"。
```json
 "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node node_modules/react-native/local-cli/cli.js start"
  }
```
其中的test节点是自动生成的，我们可以把它删除，最后我的package.json为：
```json
{
  "name": "reactnativedemo",
  "version": "1.0.0",
  "description": "",
  "main": "index.android.js",
  "scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "react": "^15.4.1",
    "react-native": "^0.38.0"
  }
}
```

### npm install --save react react-native
![图片](/images/ReactNative/React_Native_to_Native/3.png "图片")  
![图片](/images/ReactNative/React_Native_to_Native/4.png "图片")  
![图片](/images/ReactNative/React_Native_to_Native/5.png "图片")  

### curl -o .flowconfig https://raw.githubusercontent.com/facebook/react-native/master/.flowconfig  

curl是利用URL语法在命令行方式下工作的开源文件传输工具。它被广泛应用在Unix、多种Linux发行版中，并且有DOS和Win32、Win64下的移植版本。
所以可知上面这句话的意思是在对应网址下下载.flowconfig文件。  
在windows下我们要使用curl命令会提示:curl不是内部和外部命令，也不是可执行文件或批处理命令。。。
我们在windows下要使用curl命令比较麻烦。解决方法就是我们用下载工具从https://raw.githubusercontent.com/facebook/react-native/master/.flowconfig  上把.flowconfig下载下来复制到项目根目录，或者是在项目根目录下新建一个.flowconfig文件用浏览器访问这个网址其中的内容把其中的内容复制到文件当中。

## 建立index.android.js文件
在项目的根目录建立index.android.js文件并把下面的代码复制进去：  

```JavaScript
'use strict';

import React from 'react';
import {
  AppRegistry,
  StyleSheet,
  Text,
  View
} from 'react-native';

class Root extends React.Component {
  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.hello}>Welcome!</Text>
      </View>
    )
  }
}
var styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
  },
  hello: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
});

AppRegistry.registerComponent('ReactNativeView', () => Root);
```

## 添加依赖

在项目的根目录的build.gradle中加入：

```gradle
allprojects {
    repositories {
        jcenter()
        maven {
            //添加react native依赖，url路径根据实际的来写，本文的如下：
            url "$rootDir/node_modules/react-native/android"
        }
    }
}
```
注意：Android项目默认的依赖包的源为jcenter()，其中并不包含最新版的 React Native（它只到0.20.1）。
新版的React Native只在npm里发布，所以你需要增加一下依赖包的源。在编译完后，检查项目External Libraries的
react-native版本如果为0.20.1，则说明maven的依赖源没有添加成功。这时候应该是maven的路径出问题了，你要检查
路径是否正确，正确的结果为：  
![图片](/images/ReactNative/React_Native_to_Native/6.png "图片")  

在项目的模块(app)中的build.gradle文件中添加：  
文件头添加（可选）：
```
apply from: "$rootDir/node_modules/react-native/react.gradle"
```
无法编译通过的时候可以尝试添加上面这句。
```
 dependencies {
     ...
     compile "com.facebook.react:react-native:+"
 }
```
如果你想总是使用一个特定的版本，你需要把+替换成你已经下载的React Native的版本号，
这个版本号应该与package.json中的react-native的版本号("react-native": "^0.38.0")一致的。如本例中的0.38.0：  
```
 dependencies {
     ...
     compile "com.facebook.react:react-native:0.38.0"
 }
```

## 添加原生Activity文件：

### 官方教程的写法：

> MyReactActivity  

```java
import android.app.Activity;
import android.os.Bundle;
import android.view.KeyEvent;
import com.facebook.react.BuildConfig;
import com.facebook.react.ReactInstanceManager;
import com.facebook.react.ReactRootView;
import com.facebook.react.common.LifecycleState;
import com.facebook.react.modules.core.DefaultHardwareBackBtnHandler;
import com.facebook.react.shell.MainReactPackage;

public class MyReactActivity extends Activity implements DefaultHardwareBackBtnHandler {
    private ReactRootView mReactRootView;
    private ReactInstanceManager mReactInstanceManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        mReactRootView = new ReactRootView(this);
        mReactInstanceManager = ReactInstanceManager.builder()
                .setApplication(getApplication())
                .setBundleAssetName("index.android.bundle")
                .setJSMainModuleName("index.android") //对应index.android.js
                .addPackage(new MainReactPackage())
                //.setUseDeveloperSupport(BuildConfig.DEBUG) //开发者支持，BuildConfig.DEBUG的值默认是false，无法使用开发者菜单
                .setUseDeveloperSupport(true) //开发者支持,开发的时候要设置为true，不然无法使用开发者菜单
                .setInitialLifecycleState(LifecycleState.RESUMED)
                .build();
        //这里的ReactNativeView对应index.android.js中AppRegistry.registerComponent('ReactNativeView', () => ReactNativeView)的ReactNativeView
        mReactRootView.startReactApplication(mReactInstanceManager, "ReactNativeView", null);
        setContentView(mReactRootView);
    }

    @Override
    public void invokeDefaultOnBackPressed() {
        super.onBackPressed();
    }

    @Override
    protected void onPause() {
        super.onPause();

        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostPause(this);
        }
    }

    @Override
    protected void onResume() {
        super.onResume();

        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostResume(this, this);
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();

        if (mReactInstanceManager != null) {
            mReactInstanceManager.onHostDestroy(this);
        }
    }

    @Override
    public void onBackPressed() {
        if (mReactInstanceManager != null) {
            mReactInstanceManager.onBackPressed();
        } else {
            super.onBackPressed();
        }
    }

    @Override
    public boolean onKeyUp(int keyCode, KeyEvent event) {
        //当我们点击菜单的时候打开发者菜单，一个弹窗（此处需要悬浮窗权限才能显示）
        if (keyCode == KeyEvent.KEYCODE_MENU && mReactInstanceManager != null) {
            mReactInstanceManager.showDevOptionsDialog();
            return true;
        }
        return super.onKeyUp(keyCode, event);
    }
}
```
### <font color="#EF7A7A">注意：</font>

官方教程的写法中这里：

```java
 @Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    mReactRootView = new ReactRootView(this);
    mReactInstanceManager = ReactInstanceManager.builder()
            .setApplication(getApplication())
            .setBundleAssetName("index.android.bundle")
            .setJSMainModuleName("index.android") //对应index.android.js
            .addPackage(new MainReactPackage())
            //.setUseDeveloperSupport(BuildConfig.DEBUG) //开发者支持，BuildConfig.DEBUG的值默认是false，无法使用开发者菜单
            .setUseDeveloperSupport(true) //开发者支持,开发的时候要设置为true，不然无法使用开发者菜单
            .setInitialLifecycleState(LifecycleState.RESUMED)
            .build();
    //这里的ReactNativeView对应index.android.js中AppRegistry.registerComponent('ReactNativeView', () => ReactNativeView)的ReactNativeView
    mReactRootView.startReactApplication(mReactInstanceManager, "ReactNativeView", null);
    setContentView(mReactRootView);
}
```
的setUseDeveloperSupport(BuildConfig.DEBUG)方法，设置开发者支持，BuildConfig.DEBUG的值默认是false，无法使用开发者支持（开发者菜单、即时预览等），所以我们要把BuildConfig.DEBUG改为true。

### 另一种原生Activity写法

> MyReactNativeActivity

```java
import com.facebook.react.ReactActivity;

public class MyReactNativeActivity extends ReactActivity {
    /**
     * 这里的ReactNativeView对应index.android.js中AppRegistry.registerComponent('ReactNativeView', () => Root)的ReactNativeView
     */
    @Override
    protected String getMainComponentName() {
        return "ReactNativeView";
    }
}
```

### 两种原生Activity写法的对比：

##### 第一种（官方例子的）

这种写法的优势是可以利用React Native来写我们界面中的某一块区域，就是利用原生布局的addView()方法把mReactRootView加入到布局中，比如：

> activity_my_react.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="200dp"
        android:background="#9DB16D"
        android:textSize="40sp"
        android:text="原生控件TextView" />

    <LinearLayout
        android:id="@+id/layout"
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</LinearLayout>
```

> MyReactActivity修改

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_my_react);
    mReactLayout = (LinearLayout) findViewById(R.id.layout);

    mReactRootView = new ReactRootView(this);
    mReactInstanceManager = ReactInstanceManager.builder()
            .setApplication(getApplication())
            .setBundleAssetName("index.android.bundle")
            .setJSMainModuleName("index.android")//对应index.android.js
            .addPackage(new MainReactPackage())
            //.setUseDeveloperSupport(BuildConfig.DEBUG) //开发者支持，BuildConfig.DEBUG的值默认是false，无法使用开发者菜单
            .setUseDeveloperSupport(true) //开发者支持,开发的时候要设置为true，不然无法使用开发者菜单
            .setInitialLifecycleState(LifecycleState.RESUMED)
            .build();
    //这里的ReactNativeView对应index.android.js中AppRegistry.registerComponent('ReactNativeView', () => Root)的ReactNativeView
    mReactRootView.startReactApplication(mReactInstanceManager, "ReactNativeView", null);
    mReactLayout.addView(mReactRootView);
}
```

> index.android.js

```js
import React, { Component } from 'react';
import {
  View,
  Text,
  StyleSheet
} from 'react-native'

export default class Root extends Component {
  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.welcome}>React Native组件</Text>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#65A35F',
  },
  welcome: {
    fontSize: 40,
    textAlign: 'center',
    margin: 10,
  }
});

AppRegistry.registerComponent('ReactNativeView', () => Root);
```

#### 第二种

这种方式是优势是写法简单。但是无法局部使用React Native来布局。

## AndroidManifest.xml相关

> AndroidManifest.xml  

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="top.cokernut.reactnativetonative">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>

    <application
        android:allowBackup="true"
        android:name=".MyApplication"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <activity
            android:name=".MyReactActivity"
            android:label="MyReactActivity">
        </activity>
        <activity
            android:name=".MyReactNativeActivity"
            android:label="MyReactNativeActivity"
            android:theme="@style/Theme.AppCompat.Light.NoActionBar">
        </activity>
        <activity android:name="com.facebook.react.devsupport.DevSettingsActivity" />
    </application>

</manifest>
```
网络权限：
```
<uses-permission android:name="android.permission.INTERNET" />
```
悬浮窗权限：
```
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
```
有悬浮窗权限才能显示：  
![图片](/images/ReactNative/React_Native_to_Native/7.png "图片")    

如果遇到React Native的一些组件不能使用可以尝试在注册Activity时添加主题为Theme.AppCompat.Light.NoActionBar，看能否解决问题，因为一些组件依赖于这个主题:
```
<activity
    android:name=".MyReactNativeActivity"
    android:label="@string/app_name"
    android:theme="@style/Theme.AppCompat.Light.NoActionBar">
</activity>
```
开发设置界面：  
![图片](/images/ReactNative/React_Native_to_Native/8.png "图片")     

``` 
<activity android:name="com.facebook.react.devsupport.DevSettingsActivity" />
```

## 运行应用

### 启动开发服务器

在项目的根目录下运行：
> npm start  

![图片](/images/ReactNative/React_Native_to_Native/9.png "图片")     

这个命令运行的是我们package.json中配置的：
```
"scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start"
  },
```
可以用浏览器访问http://localhost:8081/index.android.bundle?platform=android 看看是否可以看到打包后的脚本。
第一次访问通常需要十几秒，并且在命令行可以看到进度条。

### 构建与运行你的程序

两种方法：  
1. 直接利用Android Studio像平常一样运行项目  
2. 在命令行中项目目录下运行gradlew installDebug  

如果你使用的是Android Studio为你构建而不是Gradle构建(gradlew installDebug)，你要确保你在安装应用之前运行了npm start，以防止它们之间出现冲突。  
效果：  
![图片](/images/ReactNative/React_Native_to_Native/15.png "图片")

MyReactActivity:  
![图片](/images/ReactNative/React_Native_to_Native/16.png "图片") ![图片](/images/ReactNative/React_Native_to_Native/17.png "图片")   

MyReactNativeActivity:  
![图片](/images/ReactNative/React_Native_to_Native/18.png "图片") ![图片](/images/ReactNative/React_Native_to_Native/19.png "图片")   

### 在Android Studio中打包成独立安装程序（release）

你可以使用Android Studio来创建你的App的发布版本！像以前创建原生应用程序的发布版本一样简单，只是有一个额外步骤：
在你打包你的发布版本之前要创建一个bundle文件，这个bundle文件会创建在项目的assets目录中，并且这个文件会包含在你的apk包中，
在你的项目根目录中运行：  
> react-native bundle --platform android --dev false --entry-file index.android.js --bundle-output app/src/main/assets/index.android.bundle --assets-dest app/src/main/res/  

app/src/main根据实际情况改为自己项目中的目录，参考assets文件夹的目录。  
![图片](/images/ReactNative/React_Native_to_Native/11.png "图片")  
结果为：  
![图片](/images/ReactNative/React_Native_to_Native/10.png "图片")   
如果报错： 
> ENOENT: no such file or directory  

你需要在你的app模块中建立assets文件夹和index.android.bundle。

现在你可以对你的应用程序进行打包发布了。

#### debug模式release模式React Native JS代码调试的区别:

debug模式: 修改完js代码打开开发者菜单点击Reload就可以看到更新后的效果，或者是开启Live Reload(点击Enable Live Reload)
这样我们修改了js文件只要保存就会自动Reload。
release模式: 修改完js代码需要重新生成index.android.bundle 文件，点击run之后才能看到效果。因为正式版发布后是无法
依赖本地服务器去更新index.android.bundle，需要把index.android.bundle打包到apk中才能运行。

## 更新React Naive版本

1.打开项目目录下的package.json文件，然后在dependencies模块下找到react-native，将当前版本号改到最新（或指定）版本号，如：

```json
{
"name": "reactnativedemo",
"version": "1.0.0",
"description": "",
"main": "index.android.js",
"scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start"
},
"author": "",
"license": "ISC",
"dependencies": {
    "react": "^15.4.1",
    "react-native": "^0.38.0"
}
}
```
react-native的npm包的最新版本可以去[这里](https://www.npmjs.com/package/react-native)查看，或使用npm info react-native命令查看。

2.项目的根目录执行：

> npm install

安装最新的React Native版本,成功后可能会出现如下类似警告：

> npm WARN react-native@0.38.0 requires a peer of react@15.4.1 but none was installed.  

3.根据警告执行：

> npm install –save react@15.4.1

更新最新的React且项目下package.json 的 dependencies下的react版本会被修改为 15.4.1

4.新版本的npm包通常还会包含一些动态生成的文件，这些文件是在运行react-native init创建新
项目时生成的，比如iOS和Android的项目文件。为了使老项目的项目文件也能得到更新
（不重新init），你需要在命令行中运行：

> react-native upgrade  

这一命令会检查最新的项目模板，然后进行如下操作：

+ 如果是新添加的文件，则直接创建。
+ 如果文件和当前版本的文件相同，则跳过。
+ 如果文件和当前版本的文件不同，则会提示你一些选项：查看两者的不同，选择保留你的版本或是用新的模板覆盖。你可以按下h键来查看所有可以使用的命令。

注意：如果你有修改原生代码，那么在使用upgrade升级前，先备份，再覆盖。覆盖完成后，使用比对工具找出差异，将你之前修改的代码逐步搬运到新文件中。

5.执行：

> react-native -v

通过如上命令来看最新的版本，检测是否升级成功！

## 问题与解决方案

### 打不开开发者菜单
查看AndroidManifest.xml文件中是否加入了悬浮窗权限：  
```
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
```
并且你要在手机上开启你要调试的应用的悬浮窗权限，不同的手机开启方式不同，可以自行搜索开启方法。

### 错误 "/data/data/package-name/lib-main/libgnustl_shared.so" is 32-bit instead of 64-bit
取消掉所有的64位的.so文件，全部加载32位的.so文件。  

1. 在项目的根目录的build.gradle中加入：   
> android.useDeprecatedNdk=true.  

2. 在项目的模块(app)中的build.gradle文件中添加:

```json
android {
    ...
    defaultConfig {
        ...
        ndk {
            abiFilters "armeabi-v7a", "x86"
        }

        packagingOptions {
            exclude "lib/arm64-v8a/librealm-jni.so"
        }
    }
}
```

### 错误：Could not get BatchedBridge,make sure your bundle is packaged correctly

1. build模式选择了release模式，引发的这个错误，你可以检查一下是否是debug模式，如果不是改为debug再试一下。
2. ReactInstanceManager的setUseDeveloperSupport(BuildConfig.DEBUG)方法值是否正确，设置开发者支持，BuildConfig.DEBUG的值默认是false，无法使用开发者支持（开发者菜单、即时预览等），所以我们要把BuildConfig.DEBUG改为true。

### 错误
![图片](/images/ReactNative/React_Native_to_Native/12.png "图片")   
#### 解决方法：    
把Android Studio自动生成的文件夹androidTest和test删除，并修改项目的模块(app)的build.gradle文件：  
#### 修改前：  
![图片](/images/ReactNative/React_Native_to_Native/13.png "图片")  
#### 修改后：  
![图片](/images/ReactNative/React_Native_to_Native/14.png "图片")     


### 其他问题可以参考[React Native for Android Windows环境搭建](http://cokernut.top/2016/11/23/Android/React%20Native%20for%20Android%20Windows%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/)    

<font size=5>[源代码](https://github.com/cokernut/ReactNativeToNative)</font>