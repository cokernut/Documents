---
title: React Native使用Android原生模块
date: 2016-12-11
tags: [Android, React Native]
categories: React Native
description:
---

React Native使用Android原生模块
<!--more-->

有时候App需要访问平台API，但React Native可能还没有相应的模块包装；或者你需要复用一些Java代码，而不是用Javascript重新实现一遍；又或者你需要实现某些高性能的、多线程的代码，譬如图片处理、数据库、或者各种高级扩展等等。

React Native可以在基础上编写真正的原生代码，并且可以访问平台所有的能力。这是一个相对高级的特性，如果React Native还不支持某个你需要的原生特性，你应当可以自己实现该特性的封装。

## Toast模块

### 创建模块

我们以Toast模块为例子，假设我们希望可以从Javascript发起一个Toast消息（Android中的一种会在屏幕下方弹出、保持一段时间的消息通知）

我们首先来创建一个原生模块。一个原生模块是一个继承了ReactContextBaseJavaModule的Java类，它可以实现一些JavaScript所需的功能。我们这里的目标是可以在JavaScript里写`AndroidToast.show('使用原生模块包装的Toast', AndroidToast.SHORT);`来调起一个Toast通知。

```java
import android.widget.Toast;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;

import java.util.HashMap;
import java.util.Map;

/**
 * ToastModule原生模块
 */
public class ToastModule extends ReactContextBaseJavaModule {

  private static final String DURATION_SHORT_KEY = "SHORT";
  private static final String DURATION_LONG_KEY = "LONG";

  public ToastModule(ReactApplicationContext reactContext) {
    super(reactContext);
  }

  /**
   * @return 返回一个字符串名字，这个名字在JavaScript端标记这个模块,这里我们把这个模块叫做AndroidToast，
   * 这样就可以在JavaScript中通过React.NativeModules.AndroidToast访问到这个模块
   *
   * 注意：React Native已经内置了一个名为ToastAndroid的模块,名字不能返回ToastAndroid，否则运行会报错
   */
  @Override
  public String getName() {
    return "AndroidToast";
  }

  /**
   * @return 返回了需要导出给JavaScript使用的常量, 比如：AndroidToast.show('使用原生模块包装的Toast', AndroidToast.SHORT);
   * 中的AndroidToast.SHORT，SHORT对应的是常量DURATION_SHORT_KEY的值"SHORT",在原生端得到的值为Toast.LENGTH_SHORT这个常量的值0
   */
  @Override
  public Map<String, Object> getConstants() {
    final Map<String, Object> constants = new HashMap<>();
    constants.put(DURATION_SHORT_KEY, Toast.LENGTH_SHORT);
    constants.put(DURATION_LONG_KEY, Toast.LENGTH_LONG);
    return constants;
  }

  /**
   * 导出给JavaScript使用的方法，Java方法需要使用注解@ReactMethod。方法的返回类型必须为void。
   * 比如：AndroidToast.show('使用原生模块包装的Toast', AndroidToast.SHORT);
   * @param message 显示内容
   * @param duration 显示时间参数
   */
  @ReactMethod
  public void show(String message, int duration) {
    Toast.makeText(getReactApplicationContext(), message, duration).show();
  }
}
```

ReactContextBaseJavaModule要求派生类实现getName方法。这个函数用于返回一个字符串名字，这个名字在JavaScript端标记这个模块。这里我们把这个模块叫做AndroidToast，这样就可以在JavaScript中通过React.NativeModules.AndroidToast访问到这个模块。
注意：React Native已经内置了一个名为ToastAndroid的模块，名字不能返回ToastAndroid，否则运行时会报错名字冲突！

```java
@Override
 public String getName() {
   return "AndroidToast";
 }
```

注意：模块名前的RCT前缀会被自动移除。所以如果返回的字符串为"RCTAndroidToast"，在JavaScript端依然通过React.NativeModules.AndroidToast访问到这个模块。

一个可选的方法getContants返回了需要导出给JavaScript使用的常量。它并不一定需要实现，但在定义一些可以被JavaScript同步访问到的预定义的值时非常有用。

```java
@Override
public Map getConstants() {
  final Map constants = new HashMap<>();
  constants.put(DURATION_SHORT_KEY, Toast.LENGTH_SHORT);
  constants.put(DURATION_LONG_KEY, Toast.LENGTH_LONG);
  return constants;
}
```

要导出一个方法给JavaScript使用，Java方法需要使用注解@ReactMethod。方法的返回类型必须为void。React Native的跨语言访问是异步进行的，所以想要给JavaScript返回一个值的唯一办法是使用回调函数或者发送事件。

```java
@ReactMethod
public void show(String message, int duration) {
  Toast.makeText(getReactApplicationContext(), message, duration).show();
}
```

### 参数类型

下面的参数类型在@ReactMethod注明的方法中，会被直接映射到它们对应的JavaScript类型。

Boolean -> Bool
Integer -> Number
Double -> Number
Float -> Number
String -> String
Callback -> function
ReadableMap -> Object
ReadableArray -> Array

### 注册模块

在Java这边要做的最后一件事就是注册这个模块。我们需要在应用的Package类的createNativeModules方法中添加这个模块。如果模块没有被注册，它也无法在JavaScript中被访问到。

```java
import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.JavaScriptModule;
import com.facebook.react.uimanager.ViewManager;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;

import java.util.List;
import java.util.ArrayList;
import java.util.Collections;

import top.cokernut.reactnativetonative.modules.ToastModule;
import top.cokernut.reactnativetonative.reactview.TextViewManager;

public class ReactNativePackage implements ReactPackage {

  @Override
  public List<Class<? extends JavaScriptModule>> createJSModules() {
    return Collections.emptyList();
  }

  @Override
  public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
    return Collections.emptyList();
  }

  @Override
  public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
    List<NativeModule> modules = new ArrayList<>();
    modules.add(new ToastModule(reactContext));
    return modules;
  }
}
```

这个package需要在项目的自定义的Application类的getPackages方法中提供。如果没有自定义的Application类则需要新建这个类，并且这个类实现了ReactApplication接口。

```java
import android.app.Application;
import com.facebook.react.ReactApplication;
import com.facebook.react.ReactNativeHost;
import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.shell.MainReactPackage;
import com.facebook.react.uimanager.ViewManager;
import com.facebook.soloader.SoLoader;

import java.util.Arrays;
import java.util.List;

public class MyApplication extends Application implements ReactApplication {
    private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
        @Override
        protected boolean getUseDeveloperSupport() {
            return BuildConfig.DEBUG;
        }

        @Override
        protected List<ReactPackage> getPackages() {
            return Arrays.<ReactPackage>asList(
                    new MainReactPackage(),
                    new ReactNativePackage()); // <-- 添加这一行，类名替换成你的Package类的名字.
        }
    };

    @Override
    public ReactNativeHost getReactNativeHost() {
        return mReactNativeHost;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        SoLoader.init(this, /* native exopackage */ false);
    }
}
```

注意：别忘记修改AndroidManifest文件的application节点的name为我们自定义的Application（如：MyApplication ）：

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
        ...
    </application>

</manifest>
```

为了让你的功能从JavaScript端访问起来更为方便，通常我们都会把原生模块封装成一个JavaScript模块。这不是必须的，但省下了每次都从NativeModules中获取对应模块的步骤。这个JS文件也可以用于添加一些其他JavaScript端实现的功能。

> AndroidToast.js

```js
import { NativeModules } from 'react-native';
module.exports = NativeModules.AndroidToast;// 下一句中的AndroidToast即对应上文 public String getName()中返回的字符串
```

> 调用AndroidToast

```js
import React, { Component } from 'react';
import {
  Text,
  TouchableOpacity
} from 'react-native'

import AndroidToast from './AndroidToast'

class Sample extends Component { 
  _press() {
    //调用原生端写的show（@ReactMethod注解的方法）方法，并把需要的值传过去
    AndroidToast.show('使用原生模块包装的Toast', AndroidToast.SHORT);
  }

  render() {
    return (
      <TouchableOpacity onPress={this._press}>
        <Text>Toast模块</Text>
      </TouchableOpacity>
    );
  }
}

export default Sample;
```

### 回调函数

原生模块还支持一种特殊的参数——回调函数。它提供了一个函数来把返回值传回给JavaScript。

```java
/**
 * ToastModule原生模块
 */
public class ToastModule extends ReactContextBaseJavaModule {
  ...
 /**
   * 导出给JavaScript使用的方法，Java方法需要使用注解@ReactMethod。方法的返回类型必须为void。
   * 比如：AndroidToast.show('使用原生模块包装的Toast', AndroidToast.SHORT);
   * @param message 显示内容
   * @param duration 显示时间参数
   * @param callback 回调函数
   */
  @ReactMethod
  public void show(String message, int duration, Callback callback) {
    Toast.makeText(getReactApplicationContext(), message, duration).show();
    callback.invoke(message, "调用成功", 123); //调用回调函数
  }
}
```

这个函数可以在JavaScript里这样使用：

```js
...
class Sample extends Component { 
  _press() {
    //调用原生端写的show（@ReactMethod注解的方法）方法，并把需要的值传过去
    AndroidToast.show('使用原生模块包装的Toast', AndroidToast.SHORT, 
      (msg, str, num) => { //回调函数，为了试验传值，所以这里是先把n传到原生端，在原生端接收到之后当作次数（num）传回到JavaScript端接收
        console.log(msg + str + num);
      });
  }
...
export default Sample;
```

原生模块通常只应调用回调函数一次。但是，它可以保存callback并在将来调用。

请务必注意callback并非在对应的原生函数返回后立即被执行——注意跨语言通讯是异步的，这个执行过程会通过消息循环来进行。

### Promises

原生模块还可以使用promise来简化代码，搭配ES2016(ES7)标准的async/await语法则效果更佳。如果桥接原生方法的最后一个参数是一个Promise，则对应的JS方法就会返回一个Promise对象。

我们对ToastModule进行改写，主要加了showTwo方法（注意不能使用重载，要取不同的方法名，不然会报错）：

```java
import com.facebook.react.bridge.Arguments;
import com.facebook.react.bridge.Callback;
import com.facebook.react.bridge.Promise;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.WritableMap;

import java.util.HashMap;
import java.util.Map;

/**
 * ToastModule原生模块
 */
public class ToastModule extends ReactContextBaseJavaModule {

  private static final String DURATION_SHORT_KEY = "SHORT";
  private static final String DURATION_LONG_KEY = "LONG";

  public ToastModule(ReactApplicationContext reactContext) {
    super(reactContext);
  }

  /**
   * @return 返回一个字符串名字，这个名字在JavaScript端标记这个模块,这里我们把这个模块叫做AndroidToast，
   * 这样就可以在JavaScript中通过React.NativeModules.AndroidToast访问到这个模块
   *
   * 注意：React Native已经内置了一个名为ToastAndroid的模块,名字不能返回ToastAndroid，否则运行会报错
   */
  @Override
  public String getName() {
    return "AndroidToast";
  }

  /**
   * @return 返回了需要导出给JavaScript使用的常量, 比如：AndroidToast.show('使用原生模块包装的Toast', AndroidToast.SHORT);
   * 中的AndroidToast.SHORT，SHORT对应的是常量DURATION_SHORT_KEY的值"SHORT",在原生端得到的值为Toast.LENGTH_SHORT这个常量的值0
   */
  @Override
  public Map<String, Object> getConstants() {
    final Map<String, Object> constants = new HashMap<>();
    constants.put(DURATION_SHORT_KEY, Toast.LENGTH_SHORT);
    constants.put(DURATION_LONG_KEY, Toast.LENGTH_LONG);
    return constants;
  }

  /**
   * 导出给JavaScript使用的方法，Java方法需要使用注解@ReactMethod。方法的返回类型必须为void。
   * 调用如：AndroidToast.show('使用原生模块包装的Toast', AndroidToast.SHORT, (msg, str, num) => {console.log(msg + str + num);}););
   * @param message 显示内容
   * @param duration 显示时间参数
   * @param callback 回调函数
   */
  @ReactMethod
  public void show(String message, int duration, Callback callback) {
    Toast.makeText(getReactApplicationContext(), message, duration).show();
    callback.invoke(message, "调用成功", 123); //调用回调函数
  }

  /**
   * 导出给JavaScript使用的方法，Java方法需要使用注解@ReactMethod。方法的返回类型必须为void。
   * 调用如：AndroidToast.show('使用原生模块包装的Toast', AndroidToast.SHORT);
   * @param message 显示内容
   * @param duration 显示时间参数
   * @param promise Promise
   */
  @ReactMethod
  public void showTwo(String message, int duration, Promise promise) {
    Toast.makeText(getReactApplicationContext(), message, duration).show();
    WritableMap map = Arguments.createMap();
    map.putString("name", "Tom");
    map.putInt("age", 20);
    promise.resolve(map);
  }
}
```

现在JavaScript端的方法会返回一个Promise。这样你就可以在一个声明了async的异步函数内使用await关键字来调用，并等待其结果返回。（虽然这样写着看起来像同步操作，但实际仍然是异步的，并不会阻塞执行来等待）。

```javascript
import React, { Component } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet
} from 'react-native'

import AndroidToast from '../modules/AndroidToast'

class Sample extends Component { 
  constructor(props) {
    super(props);
    this.state = {};
    this._pressPromise = this._pressPromise.bind(this);
  }

  _press() {
    //调用原生端写的show（@ReactMethod注解的方法）方法，并把需要的值传过去
    AndroidToast.show('使用原生模块包装的Toast 回调', AndroidToast.SHORT, 
      (msg, str, num) => { //回调函数，为了试验传值，所以这里是先把n传到原生端，在原生端接收到之后当作次数（num）传回到JavaScript端接收
        console.log(msg + str + num);
      });
  }

  async run() {
    var {
      name,
      age
    } = await AndroidToast.showTwo('使用原生模块包装的Toast Promise', AndroidToast.SHORT); //调用原生端写的show（@ReactMethod注解的方法）方法，并把需要的值传过去
    console.log('name' + ':' + name + '\n' + 'age' + ':' + age);
  }

  _pressPromise() {
    this.run();
  }

  render() {
    return (
      <View style={styles.container}>
        <TouchableOpacity style={styles.touch} onPress={this._press}>
          <Text style={styles.text}>Toast模块回调传值</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.touch} onPress={this._pressPromise}>
          <Text style={styles.text}>Toast模块Promise传值</Text>
        </TouchableOpacity>
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
  touch: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#65A35F',
  },
  text: {
    fontSize: 40,
    textAlign: 'center',
    margin: 10,
  }
});
export default Sample;
```

### 多线程

原生模块不应对自己被调用时所处的线程做任何假设，当前的状况有可能会在将来的版本中改变。如果一个过程要阻塞执行一段时间，这个工作应当分配到一个内部管理的工作线程，然后从那边可以调用任意的回调函数。我们通常用AsyncTask来完成这项工作。

### 发送事件到JavaScript

原生模块可以在没有被调用的情况下往JavaScript发送事件通知。最简单的办法就是通过RCTDeviceEventEmitter，这可以通过ReactContext来获得对应的引用，像这样：

```java
...
private void sendEvent(ReactContext reactContext,
                       String eventName,
                       @Nullable WritableMap params) {
  reactContext
      .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
      .emit(eventName, params);
}
...
WritableMap params = Arguments.createMap();
...
sendEvent(reactContext, "keyboardWillShow", params);
```

JavaScript模块可以使用Subscribable Mixin的addListenerOn方法来监听事件：

```javascript
import { DeviceEventEmitter } from 'react-native';
...
var ScrollResponderMixin = {
  mixins: [Subscribable.Mixin],
  componentWillMount: function() {
    ...
    this.addListenerOn(DeviceEventEmitter,
                       'keyboardWillShow',
                       this.scrollResponderKeyboardWillShow);
    ...
  },
  scrollResponderKeyboardWillShow:function(e: Event) {
    this.keyboardWillOpenTo = e;
    this.props.onKeyboardWillShow && this.props.onKeyboardWillShow(e);
  },
 ```
 
你也可以通过使用DeviceEventEmitter模块来监听事件：

```javascript
...
componentWillMount: function() {
  DeviceEventEmitter.addListener('keyboardWillShow', function(e: Event) {
    // handle event.
  });
}
...
```

### 从startActivityForResult中获取结果

如果你使用startActivityForResult调起了一个activity并想从其中获取返回结果，那么你需要监听onActivityResult事件。具体的做法是继承BaseActivityEventListener或是实现ActivityEventListener。我们推荐前一种做法，因为它相对来说不太会受到API变更的影响。然后你需要在模块的构造函数中注册这一监听事件。

> reactContext.addActivityEventListener(mActivityResultListener);

现在你可以通过重写下面的方法来实现对onActivityResult的监听：

```java
@Override
public void onActivityResult(
  final Activity activity,
  final int requestCode,
  final int resultCode,
  final Intent intent) {
  // 在这里实现你自己的逻辑
}
```

下面我们写一个简单的图片选择器来实践一下。这个图片选择器会把pickImage方法暴露给JavaScript，而这个方法在调用时就会把图片的路径返回到JS端。

```java
public class ImagePickerModule extends ReactContextBaseJavaModule {

  private static final int IMAGE_PICKER_REQUEST = 467081;
  private static final String E_ACTIVITY_DOES_NOT_EXIST = "E_ACTIVITY_DOES_NOT_EXIST";
  private static final String E_PICKER_CANCELLED = "E_PICKER_CANCELLED";
  private static final String E_FAILED_TO_SHOW_PICKER = "E_FAILED_TO_SHOW_PICKER";
  private static final String E_NO_IMAGE_DATA_FOUND = "E_NO_IMAGE_DATA_FOUND";

  private Promise mPickerPromise;

  private final ActivityEventListener mActivityEventListener = new BaseActivityEventListener() {

    @Override
    public void onActivityResult(Activity activity, int requestCode, int resultCode, Intent data) {
      if (requestCode == IMAGE_PICKER_REQUEST) {
        if (mPickerPromise != null) {
          if (resultCode == Activity.RESULT_CANCELED) {
            mPickerPromise.reject(E_PICKER_CANCELLED, "Image picker was cancelled");
          } else if (resultCode == Activity.RESULT_OK) {
            Uri uri = intent.getData();

            if (uri == null) {
              mPickerPromise.reject(E_NO_IMAGE_DATA_FOUND, "No image data found");
            } else {
              mPickerPromise.resolve(uri.toString());
            }
          }

          mPickerPromise = null;
        }
      }
    }
  };

  public ImagePickerModule(ReactApplicationContext reactContext) {
    super(reactContext);

    // Add the listener for `onActivityResult`
    reactContext.addActivityEventListener(mActivityEventListener);
  }

  @Override
  public String getName() {
    return "ImagePickerModule";
  }

  @ReactMethod
  public void pickImage(final Promise promise) {
    Activity currentActivity = getCurrentActivity();

    if (currentActivity == null) {
      promise.reject(E_ACTIVITY_DOES_NOT_EXIST, "Activity doesn't exist");
      return;
    }

    // Store the promise to resolve/reject when picker returns data
    mPickerPromise = promise;

    try {
      final Intent galleryIntent = new Intent(Intent.ACTION_PICK);

      galleryIntent.setType("image/*");

      final Intent chooserIntent = Intent.createChooser(galleryIntent, "Pick an image");

      currentActivity.startActivityForResult(chooserIntent, IMAGE_PICKER_REQUEST);
    } catch (Exception e) {
      mPickerPromise.reject(E_FAILED_TO_SHOW_PICKER, e);
      mPickerPromise = null;
    }
  }
}
```

### 监听生命周期事件

监听activity的生命周期事件（比如onResume, onPause等等）和我们在前面实现 ActivityEventListener的做法类似。模块必须实现LifecycleEventListener，然后需要在构造函数中注册一个监听函数：

> reactContext.addLifecycleEventListener(this);

现在你可以通过实现下列方法来监听activity的生命周期事件了：

```java
@Override
public void onHostResume() {
    // Activity `onResume`
}

@Override
public void onHostPause() {
    // Activity `onPause`
}

@Override
public void onHostDestroy() {
    // Activity `onDestroy`
}
```