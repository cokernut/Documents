---
title: React Native使用Android原生UI控件
date: 2016-12-14
tags: [Android, React Native]
categories: React Native
description:
---

React Native使用Android原生UI控件
<!--more-->

在如今的App中，已经有成千上万的原生UI部件了——其中的一些是平台的一部分，另一些可能来自于一些第三方库，而且可能你自己还收藏了很多。React Native已经封装了大部分最常见的组件，譬如ScrollView和TextInput，但不可能封装全部组件。而且，说不定你曾经为自己以前的App还封装过一些组件，React Native肯定没法包含它们。幸运的是，在React Naitve应用程序中封装和植入已有的组件非常简单。

## TextView控件

我们以TextView控件为例子。我们为了让JavaScript中可以使用TextView，原生视图需要被一个ViewManager的派生类（或者更常见的，SimpleViewManage的派生类）创建和管理。一个SimpleViewManager可以用于这个场景，是因为它能够包含更多公共的属性，譬如背景颜色、透明度、Flexbox布局等等。

这些子类本质上都是单例——React Native只会为每个管理器创建一个实例。它们创建原生的视图并提供给NativeViewHierarchyManager，NativeViewHierarchyManager则会反过来委托它们在需要的时候去设置和更新视图的属性。ViewManager还会代理视图的所有委托，并给JavaScript发回对应的事件。

提供原生视图很简单：

1. 创建一个ViewManager的子类。
2. 实现createViewInstance方法。
3. 导出视图的属性设置器：使用@ReactProp（或@ReactPropGroup）注解。
4. 把这个视图管理类注册到应用程序包的createViewManagers里。
5. 实现JavaScript模块。

### 创建ViewManager的子类

创建一个视图管理类ReactImageManager，它继承自SimpleViewManager\<ReactImageView\>。ReactImageView是这个视图管理类所管理的对象类型，这应当是一个自定义的原生视图。getName方法返回的名字会用于在JavaScript端引用这个原生视图类型。

```java
import android.widget.TextView;

import com.facebook.react.uimanager.SimpleViewManager;
import com.facebook.react.uimanager.ThemedReactContext;
import com.facebook.react.uimanager.annotations.ReactProp;

/**
 * TextView的创建和管理
 */
public class TextViewManager extends SimpleViewManager<TextView> {
    public static final String REACT_VIEW = "TextView";

    /**
     * @return 返回的名字会用于在JavaScript端引用这个原生视图类型。
     */
    @Override
    public String getName() {
        return REACT_VIEW;
    }

    /**
     * 创建视图，且应当把自己初始化为默认的状态。所有属性的设置都通过后续的updateView来进行。
     */
    @Override
    public TextView createViewInstance(ThemedReactContext reactContext) {
        return new TextView(reactContext);
    }

    /**
     * 要导出给JavaScript使用的属性，需要申明带有@ReactProp（或@ReactPropGroup）注解的设置方法。
     * 方法的第一个参数是要修改属性的视图实例，第二个参数是要设置的属性值。
     * 方法的返回值类型必须为void，而且访问控制必须被声明为public。
     * @param view 要修改属性的视图实例
     * @param text 要设置的属性值
     *
     * 注意@ReactProp注解必须包含一个字符串类型的参数name。这个参数指定了对应属性在JavaScript端的名字。
     * 除了name，@ReactProp注解还接受这些可选的参数：defaultBoolean, defaultInt, defaultFloat。
     * 这些参数必须是对应的基础类型的值（也就是boolean, int, float），这些值会被传递给setter方法，
     * 以免JavaScript端某些情况下在组件中移除了对应的属性。
     */
    @ReactProp(name = "text")
    public void setText(TextView view, String text) {
        view.setText(text);
    }

    @ReactProp(name = "textSize", defaultFloat = 20f)
    public void setTextSize(TextView view, float size) {
        view.setTextSize(size);
    }

    @ReactProp(name = "textColor", defaultInt = 0x000000)
    public void setTextColor(TextView view, int color) {
        view.setTextColor(color);
    }
}

```
 
### 实现方法createViewInstance

视图在createViewInstance中创建，且应当把自己初始化为默认的状态。所有属性的设置都通过后续的updateView来进行。

```java
/**
 * 创建视图，且应当把自己初始化为默认的状态。所有属性的设置都通过后续的updateView来进行。
 */
@Override
public TextView createViewInstance(ThemedReactContext reactContext) {
	return new TextView(reactContext);
}
```

### 通过@ReactProp（或@ReactPropGroup）注解来导出属性的设置方法。

要导出给JavaScript使用的属性，需要申明带有@ReactProp（或@ReactPropGroup）注解的设置方法。方法的第一个参数是要修改属性的视图实例，第二个参数是要设置的属性值。方法的返回值类型必须为void，而且访问控制必须被声明为public。JavaScript所得知的属性类型会由该方法第二个参数的类型来自动决定。支持的类型有：boolean, int, float, double, String, Boolean, Integer, ReadableArray, ReadableMap。

@ReactProp注解必须包含一个字符串类型的参数name。这个参数指定了对应属性在JavaScript端的名字。

除了name，@ReactProp注解还接受这些可选的参数：defaultBoolean, defaultInt, defaultFloat。这些参数必须是对应的基础类型的值（也就是boolean, int, float），这些值会被传递给setter方法，以免JavaScript端某些情况下在组件中移除了对应的属性。注意这个"默认"值只对基本类型生效，对于其他的类型而言，当对应的属性删除时，null会作为默认值提供给方法。

使用@ReactPropGroup来注解的设置方法和@ReactProp不同。请参见@ReactPropGroup注解类源代码中的文档来获取更多详情。

**注意：** 在ReactJS里，修改一个属性会引发一次对设置方法的调用。有一种修改情况是，移除掉之前设置的属性。在这种情况下设置方法也一样会被调用，并且“默认”值会被作为参数提供（对于基础类型来说可以通过defaultBoolean、defaultFloat等@ReactProp的属性提供，而对于复杂类型来说参数则会设置为null）

```java
/**
 * 要导出给JavaScript使用的属性，需要申明带有@ReactProp（或@ReactPropGroup）注解的设置方法。
 * 方法的第一个参数是要修改属性的视图实例，第二个参数是要设置的属性值。
 * 方法的返回值类型必须为void，而且访问控制必须被声明为public。
 * @param view 要修改属性的视图实例
 * @param text 要设置的属性值
 *
 * 注意@ReactProp注解必须包含一个字符串类型的参数name。这个参数指定了对应属性在JavaScript端的名字。
 * 除了name，@ReactProp注解还接受这些可选的参数：defaultBoolean, defaultInt, defaultFloat。
 * 这些参数必须是对应的基础类型的值（也就是boolean, int, float），这些值会被传递给setter方法，
 * 以免JavaScript端某些情况下在组件中移除了对应的属性。
 */
@ReactProp(name = "text")
public void setText(TextView view, String text) {
	view.setText(text);
}

@ReactProp(name = "textSize", defaultFloat = 20f)
public void setTextSize(TextView view, float size) {
	view.setTextSize(size);
}

@ReactProp(name = "textColor", defaultInt = 0x000000)
public void setTextColor(TextView view, int color) {
	view.setTextColor(color);
}
```
 
### 注册ViewManager

在Java中的最后一步就是把视图控制器注册到应用中。这和原生模块的注册方法类似，唯一的区别是我们把它放到createViewManagers方法的返回值里。

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
    List<ViewManager> views = new ArrayList<>();
    views.add(new TextViewManager());
    return views;
  }

  @Override
  public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
    return Collections.emptyList();
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

### 实现对应的JavaScript模块

整个过程的最后一步就是创建JavaScript模块并且定义Java和JavaScript之间的接口层。大部分过程都由React底层的Java和JavaScript代码来完成，你所需要做的就是通过propTypes来描述属性的类型。

> TextView.js

```javascript
import { PropTypes } from 'react';
import { requireNativeComponent, View} from 'react-native';

let iface = {
  name: 'TextView',
  propTypes: {
    text: PropTypes.string,
    textSize: PropTypes.number,
    textColor: PropTypes.number,
    ...View.propTypes, //包含默认的View的属性，如果没有这句会报‘"..." has no propType for native prop "..." of native type "..."’错误
  },
};

export default requireNativeComponent('TextView', iface);
```

requireNativeComponent通常接受两个参数，第一个参数是原生视图的名字，而第二个参数是一个描述组件接口的对象。组件接口应当声明一个友好的name，用来在调试信息中显示；组件接口还必须声明propTypes字段，用来对应到原生视图上。这个propTypes还可以用来检查用户使用View的方式是否正确。

**注意**，如果你还需要一个JavaScript组件来做一些除了指定name和propTypes以外的事情，譬如事件处理，你可以把原生组件用一个普通React组件封装。在这种情况下，requireNativeComponent的第二个参数变为用于封装的组件。

**注意**：和原生模块不同，原生视图的前缀RCT不会被自动去掉。

### 使用实现的原生组件

```javascript
import React, { Component } from 'react';
import {} from 'react-native'

import TextView from '../view/TextView'

class Sample extends Component {
  render() {
    return (
          <TextView text='TextView' textSize={40} textColor={0x00ff00} style={{width:200, height:100}}/>
    );
  }
}

export default Sample;
```

## 事件

现在我们已经知道了怎么导出一个原生视图组件，并且我们可以在JS里很方便的控制它了。不过我们怎么才能处理来自用户的事件，譬如缩放操作或者拖动？当一个原生事件发生的时候，它应该也能触发JavaScript端视图上的事件，这两个视图会依据getId()而关联在一起。

```java
class MyCustomView extends View {
   ...
   public void onReceiveNativeEvent() {
      WritableMap event = Arguments.createMap();
      event.putString("message", "MyMessage");
      ReactContext reactContext = (ReactContext)getContext();
      reactContext.getJSModule(RCTEventEmitter.class).receiveEvent(
          getId(),
          "topChange",
          event);
    }
}
```

这个事件名topChange在JavaScript端映射到onChange回调属性上（这个映射关系在UIManagerModuleConstants.java文件里）。这个回调会被原生事件执行，然后我们通常会在封装组件里构造一个类似的API：


```javascript
class MyCustomView extends React.Component {
  constructor() {
    this._onChange = this._onChange.bind(this);
  }
  _onChange(event: Event) {
    if (!this.props.onChangeMessage) {
      return;
    }
    this.props.onChangeMessage(event.nativeEvent.message);
  }
  render() {
    return <RCTMyCustomView {...this.props} onChange={this._onChange} />;
  }
}
MyCustomView.propTypes = {
  /**
   * Callback that is called continuously when the user is dragging the map.
   */
  onChangeMessage: React.PropTypes.func,
  ...
};

var RCTMyCustomView = requireNativeComponent(`RCTMyCustomView`, MyCustomView, {
  nativeOnly: {onChange: true}
});
```

注意上面用到了nativeOnly。有时候有一些特殊的属性，想从原生组件中导出，但是又不希望它们成为对应React封装组件的属性。举个例子，Switch组件可能在原生组件上有一个onChange事件，然后在封装类中导出onValueChange回调属性。这个属性在调用的时候会带上Switch的状态作为参数之一。这样的话你可能不希望原生专用的属性出现在API之中，也就不希望把它放到propTypes里。可是如果你不放的话，又会出现一个报错。解决方案就是带上nativeOnly选项。

