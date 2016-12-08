---
title: React Native学习笔记--进阶（三）--定时器、直接操作（setNativeProps）、调试
date: 2016-12-07
tags: React Native
categories: React Native
description:
---
React Native 进阶（三）--定时器、直接操作（setNativeProps）、调试
<!--more-->

## 定时器

+ setTimeout, clearTimeout
+ setInterval, clearInterval
+ setImmediate, clearImmediate
+ requestAnimationFrame, cancelAnimationFrame

requestAnimationFrame(fn)和setTimeout(fn, 0)不同，前者会在每帧刷新之后执行一次，而后者则会尽可能快的执行。

setImmediate则会在当前JavaScript执行块结束的时候执行，就在将要发送批量响应数据到原生之前。注意如果你在setImmediate的回调函数中又执行了setImmediate，它会紧接着立刻执行，而不会在调用之前等待原生代码。

Promise的实现就使用了setImmediate来执行异步调用。

### InteractionManager

原生应用感觉如此流畅的一个重要原因就是在互动和动画的过程中避免繁重的操作。在React Native里，你可以用InteractionManager将一些耗时较长的工作安排到所有互动或动画完成之后再进行，这样可以保证JavaScript动画的流畅运行。

应用可以通过以下代码来安排一个任务，使其在交互结束之后执行：

```js
InteractionManager.runAfterInteractions(() => {
   // ...需要长时间同步执行的任务...
});
```
我们来把它和之前的几个任务安排方法对比一下：

+ requestAnimationFrame(): 用来执行在一段时间内控制视图动画的代码
+ setImmediate/setTimeout/setInterval(): 在稍后执行代码。注意这有可能会延迟当前正在进行的动画。
+ runAfterInteractions(): 在稍后执行代码，不会延迟当前进行的动画。

触摸处理系统会把一个或多个进行中的触摸操作认定为'交互'，并且会将runAfterInteractions()的回调函数延迟执行，直到所有的触摸操作都结束或取消了。

InteractionManager还允许应用注册动画，在动画开始时创建一个交互“句柄”，然后在结束的时候清除它。

```js
var handle = InteractionManager.createInteractionHandle();
// 执行动画... (`runAfterInteractions`中的任务现在开始排队等候)
// 在动画完成之后
InteractionManager.clearInteractionHandle(handle);
// 在所有句柄都清除之后，现在开始依序执行队列中的任务
```

runAfterInteractions接受一个普通的回调函数，或是一个PromiseTask对象，该对象需要带有名为gen的方法，并返回一个Promise。如果提供的参数是一个PromiseTask， 那么即便它是异步的它也会阻塞任务队列，直到它（以及它所有的依赖任务，哪怕这些依赖任务也是异步的）执行完毕后，才会执行下一个任务。

默认情况下，排队的任务会在一次setImmediate方法中依序批量执行。如果你调用了setDeadLine方法并设定了一个正整数值，则任务只会在设定的时间到达后开始执行。在此之前，任务会通过setTimeout来挂起并阻塞其他任务执行，这样可以给诸如触摸交互一类的事件留出时间，使应用可以更快地响应用户。

#### 方法
> static runAfterInteractions(callback: Function) 

安排一个函数在所有的交互和动画完成之后运行。返回一个可取消的promise。

>static createInteractionHandle() 

通知管理器有某个动画或者交互开始了。

> static clearInteractionHandle(handle: Handle) 

通知管理器有某个动画或者交互已经结束了。

> static setDeadline(deadline: number) 

如果设定了一个正整数值，则会使用setTimeout来挂起所有尚未执行的任务。在eventLoopRunningTime到达设定时间后，才开始使用一个setImmediate方法来批量执行所有任务。

#### 属性

Events: CallExpression 
addListener: CallExpression 

### TimerMixin

很多React Native应用发生致命错误（闪退）是与计时器有关。具体来说，是在某个组件被卸载（unmount）之后，计时器却仍然被激活。为了解决这个问题，引入了TimerMixin。如果你在组件中引入TimerMixin，就可以把你原本的setTimeout(fn, 500)改为this.setTimeout(fn, 500)(只需要在前面加上this.)，然后当你的组件卸载时，所有的计时器事件也会被正确的清除。

这个库并没有跟着React Native一起发布。你需要在项目文件夹下输入npm i react-timer-mixin --save来单独安装它。

```js
import TimerMixin from 'react-timer-mixin';

var Component = React.createClass({
  mixins: [TimerMixin],
  componentDidMount: function() {
    this.setTimeout(
      () => { console.log('I do not leak!'); },
      500
    );
  }
});
```
强烈建议您使用react-timer-mixin提供的this.setTimeout(...)来代替setTimeout(...)。这可以规避许多难以排查的BUG。

如果你的项目是用ES6代码编写，因为ES6中没有内置Mixin，你可以使用[react-mixin](https://github.com/brigand/react-mixin)来代替TimerMixin或者在unmount组件时记住清除（clearTimeout/clearInterval）所有用到的定时器，那么也可以实现和TimerMixin同样的效果。例如：

```js
import React,{
  Component
} from 'react';

export default class Hello extends Component {
  componentDidMount() {
    this.timer = setTimeout(
      () => { console.log('把一个定时器的引用挂在this上'); },
      500
    );
  }
  componentWillUnmount() {
    // 如果存在this.timer，则使用clearTimeout清空。
    // 如果你使用多个timer，那么用多个变量，或者用个数组来保存引用，然后逐个clear
    this.timer && clearTimeout(this.timer);
  }
};
```

## 直接操作（setNativeProps）

有时候我们需要直接改动组件并触发局部的刷新，但不使用state或是props。譬如在浏览器中使用React库，有时候会需要直接修改一个DOM节点，而在手机App中操作View时也会碰到同样的情况。在React Native中，setNativeProps就是等价于直接操作DOM节点的方法。

>什么时候使用setNativeProps呢？在（不得不）频繁刷新而又遇到了性能瓶颈的时候。
直接操作组件并不是应该经常使用的工具。一般来说只是用来创建连续的动画，同时避免渲染组件结构和同步太多视图变化所带来的大量开销。setNativeProps是一个“简单粗暴”的方法，它直接在底层（DOM、UIView等）而不是React组件中记录state，这样会使代码逻辑难以理清。所以在使用这个方法之前，请尽量先尝试用setState和shouldComponentUpdate方法来解决问题。

### setNativeProps与TouchableOpacity

TouchableOpacity这个组件就在内部使用了setNativeProps方法来更新其子组件的透明度：

```js
setOpacityTo(value) {
  // Redacted: animation related code
  this.refs[CHILD_REF].setNativeProps({
    opacity: value
  });
},
```
由此我们可以写出下面这样的代码：子组件可以响应点击事件，更改自己的透明度。而子组件自身并不需要处理这件事情，也不需要在实现中做任何修改。

```js
<TouchableOpacity onPress={this._handlePress}>
  <View style={styles.button}>
    <Text>Press me!</Text>
  </View>
</TouchableOpacity>
```
如果不使用setNativeProps这个方法来实现这一需求，那么一种可能的办法是把透明值保存到state中，然后在onPress事件触发时更新这个值：

```js
constructor(props) {
  super(props);
  this.state = { myButtonOpacity: 1, };
}

render() {
  return (
    <TouchableOpacity onPress={() => this.setState({myButtonOpacity: 0.5})}
                      onPressOut={() => this.setState({myButtonOpacity: 1})}>
      <View style={[styles.button, {opacity: this.state.myButtonOpacity}]}>
        <Text>Press me!</Text>
      </View>
    </TouchableOpacity>
  )
}
```
比起之前的例子，这一做法会消耗大量的计算 —— 每一次透明值变更的时候都要重新渲染组件结构，即便视图的其他属性和子组件并没有变化。一般来说这一开销也不足为虑，但当执行连续的动画以及响应用户手势的时候，只有正确地优化组件才能提高动画的流畅度。setNativeProps方法实际是对RCTUIManager.updateView的封装 —— 而这正是重渲染所触发的函数调用。

### 复合组件与setNativeProps

复合组件并不是单纯的由一个原生视图构成，所以你不能对其直接使用setNativeProps。比如下面这个例子：

```js
class MyButton extends React.Component {
  render() {
    return (
      <View>
        <Text>{this.props.label}</Text>
      </View>
    )
  }
}

class App extends React.Component {
  render() {
    return (
      <TouchableOpacity>
        <MyButton label="Press me!" />
      </TouchableOpacity>
    )
  }
}
```

运行这个例子会马上看到一行报错： <font color="#D02591">Touchable child must either be native or forward setNativeProps to a native component.</font> 这是因为MyButton并非直接由原生视图构成，而我们只能给原生视图设置透明值。你可以尝试这样去理解：如果你通过React.createClass方法自定义了一个组件，直接给它设置样式prop是不会生效的，你得把样式props层层向下传递给子组件，直到子组件是一个能够直接定义样式的原生组件。同理，我们也需要把setNativeProps传递给由原生组件封装的子组件。

#### 将setNativeProps传递给子组件

具体要做的就是在我们的自定义组件中再封装一个setNativeProps方法，其内容为对合适的子组件调用真正的setNativeProps方法，并传递要设置的参数。

```js
class MyButton extends React.Component {
  setNativeProps(nativeProps) {
    this._root.setNativeProps(nativeProps);
  }

  render() {
    return (
      <View ref={component => this._root = component} {...this.props}>
        <Text>{this.props.label}</Text>
      </View>
    )
  }
}
```
现在你可以在TouchableOpacity中嵌入MyButton了！有一点需要特别说明：这里我们使用了ref回调语法，而不是传统的字符串型ref引用。

你可能还会注意到我们在向下传递props时使用了{...this.props}语法（对象的扩展运算符,将多个对象合并到某个对象）。这是因为TouchableOpacity本身其实也是个复合组件， 它除了要求在子组件上执行setNativeProps 以外，还要求子组件对触摸事件进行处理。因此，它会传递多个props，其中包含了onmoveshouldsetresponder 函数，这个函数需要回调给TouchableOpacity组件，以完成触摸事件的处理。与之相对的是TouchableHighlight组件，它本身是由原生视图构成，因而只需要我们实现setNativeProps。

### setNativeProps清除TextInput的值

另一种很常见的setNativeProps的用法是清除TextInput的值。当缓冲区延迟低和用户输入很快的时候可以通过控制TextInput的属性来减少输入的字符。一些开发人员更喜欢完全跳过这个属性，而直接使用setNativeProps直接操作TextInput值。
下面的代码演示了点击按钮是清除TextInput的值。

```js
class App extends React.Component {
  constructor(props) {
    super(props);
    this.clearText = this.clearText.bind(this);
  }

  clearText() {
    this._textInput.setNativeProps({text: ''});
  }

  render() {
    return (
      <View style={styles.container}>
        <TextInput ref={component => this._textInput = component}
                   style={styles.textInput} />
        <TouchableOpacity onPress={this.clearText}>
          <Text>Clear text</Text>
        </TouchableOpacity>
      </View>
    );
  }
}
```

### 避免和render方法的冲突

如果要更新一个由render方法来维护的属性，则可能会碰到一些出人意料的bug。因为每一次组件重新渲染都可能引起属性变化，这样一来，之前通过setNativeProps所设定的值就被完全忽略和覆盖掉了。

### setNativeProps与shouldComponentUpdate

通过巧妙运用 shouldComponentUpdate方法，可以避免重新渲染那些实际没有变化的子组件所带来的额外开销，此时使用setState的性能已经可以与setNativeProps相媲美了。

## 调试

### 访问App内的开发菜单

你可以通过摇晃设备或是选择iOS模拟器的"Hardware"菜单中的"Shake Gesture"选项来打开开发菜单。另外，如果是在iOS模拟器中运行，还可以按下Command⌘ + D 快捷键，Android模拟器对应的则是Command⌘ + M（windows上可能是F1或者F2）。

![开发者菜单](/images/ReactNative/advance/3.png)

> 注意：在成品（release/producation builds）中开发者菜单会被关闭。

### 刷新JavaScript

传统的原生应用开发中，每一次修改都需要重新编译，但在React Native中你只需要刷新一下JavaScript代码，就能立刻看到变化。具体的操作就是在开发菜单中点击"Reload"选项。也可以在iOS模拟器中按下Command⌘ + R ，Android模拟器上对应的则是按两下R。（注意，某些React Native版本可能在windows中reload无效，请等待官方修复）

如果在iOS模拟器中按下Command⌘ + R没啥感觉，则注意检查Hardware菜单中，Keyboard选项下的"Connect Hardware Keyboard"是否被选中。

#### 自动刷新

选择开发菜单中的"Enable Live Reload"可以开启自动刷新，这样可以节省你开发中的时间。  
更神奇的是，你还可以保持应用的当前运行状态，修改后的JavaScript文件会自动注入进来（就好比行驶中的汽车不用停下就能更换新的轮胎）。要实现这一特性只需开启开发菜单中的Hot Reloading选项。  

某些情况下hot reload并不能顺利实施。如果碰到任何界面刷新上的问题，请尝试手动完全刷新。

但有些时候你必须要重新编译应用才能使修改生效：  
增加了新的资源(比如给iOS的Images.xcassets或是Andorid的res/drawable文件夹添加了图片)
更改了任何的原生代码（objective-c/swift/java）

### 应用内的错误与警告提示（红屏和黄屏）

红屏或黄屏提示都只会在开发版本中显示，正式的离线包中是不会显示的。

#### 红屏错误
应用内的报错会以全屏红色显示在应用中（调试模式下），我们称为红屏（red box）报错。你可以使用console.error()来手动触发红屏错误。

#### 黄屏警告
应用内的警告会以全屏黄色显示在应用中（调试模式下），我们称为黄屏（yellow box）报错。点击警告可以查看详情或是忽略掉。 和红屏报警类似，你可以使用console.warn()来手动触发黄屏警告。 在默认情况下，开发模式中启用了黄屏警告。可以通过以下代码关闭：

> console.disableYellowBox = true;
> console.warn('YellowBox is disabled.');

你也可以通过代码屏蔽指定的警告，像下面这样设置一个数组：

> console.ignoredYellowBox = ['Warning: ...'];

数组中的字符串就是要屏蔽的警告的开头的内容。（例如上面的代码会屏蔽掉所有以Warning开头的警告内容）

> 红屏和黄屏在发布版（release/production）中都是自动禁用的。

### 访问控制台日志

在运行RN应用时，可以在终端中运行如下命令来查看控制台的日志：

> react-native log-ios
> react-native log-android

此外，你也可以在iOS模拟器的菜单中选择Debug → Open System Log...来查看。如果是Android应用，无论是运行在模拟器或是真机上，都可以通过在终端命令行里运行adb logcat *:S ReactNative:V ReactNativeJS:V命令来查看。

### Chrome开发者工具

在开发者菜单中选择"Debug JS Remotely"选项，即可以开始在Chrome中调试JavaScript代码。点击这个选项的同时会自动打开调试页面 http://localhost:8081/debugger-ui.

在Chrome的菜单中选择Tools → Developer Tools可以打开开发者工具，也可以通过键盘快捷键来打开（Mac上是Command⌘ + Option⌥ + I，Windows上是Ctrl + Shift + I或是F12）。打开有异常时暂停（Pause On Caught Exceptions）选项，能够获得更好的开发体验。

> Chrome中并不能直接看到App的用户界面，而只能提供console的输出，以及在sources项中断点调试js脚本。

> 目前无法正常使用React开发插件（就是某些教程或截图上提到的Chrome开发工具上多出来的React选项），但这并不影响代码的调试。如果你需要像调试web页面那样查看RN应用的jsx结构，暂时只能使用Nuclide的"React Native Inspector"这一功能来代替。

#### 使用Chrome开发者工具来在设备上调试

对于iOS真机来说，需要打开 RCTWebSocketExecutor.m文件，然后将其中的"localhost"改为你的电脑的IP地址，最后启用开发者菜单中的"Debug JS Remotely"选项。

对于Android 5.0+设备（包括模拟器）来说，将设备通过USB连接到电脑上后，可以使用adb命令行工具来设定从设备到电脑的端口转发：

> adb reverse tcp:8081 tcp:8081

如果设备Android版本在5.0以下，则可以在开发者菜单中选择"Dev Settings - Debug server host for device"，然后在其中填入电脑的”IP地址:端口“。

> 如果在Chrome调试时遇到一些问题，那有可能是某些Chrome的插件引起的。试着禁用所有的插件，然后逐个启用，以确定是否某个插件影响到了调试。

#### 使用自定义的JavaScript调试器来调试

如果想用其他的JavaScript调试器来代替Chrome，可以设置一个名为REACT_DEBUGGER的环境变量，其值为启动自定义调试器的命令。调试的流程依然是从开发者菜单中的"Debug JS Remotely"选项开始。

被指定的调试器需要知道项目所在的目录（可以一次传递多个目录参数，以空格隔开）。例如，如果你设定了REACT_DEBUGGER="node /某个路径/launchDebugger.js --port 2345 --type ReactNative"，那么启动调试器的命令就应该是node /某个路径/launchDebugger.js --port 2345 --type ReactNative /某个路径/你的RN项目目录。

> 以这种方式执行的调试器最好是一个短进程（short-lived processes），同时最好也不要有超过200k的文字输出。

#### 在Android上使用Stetho来调试

1. 在android/app/build.gradle文件中添加：

> compile 'com.facebook.stetho:stetho:1.3.1'
> compile 'com.facebook.stetho:stetho-okhttp3:1.3.1'

2. 在android/app/src/main/java/com/{yourAppName}/MainApplication.java文件中添加：

> import com.facebook.react.modules.network.ReactCookieJarContainer;
> import com.facebook.stetho.Stetho;
> import okhttp3.OkHttpClient;
> import com.facebook.react.modules.network.OkHttpClientProvider;
> import com.facebook.stetho.okhttp3.StethoInterceptor;
> import java.util.concurrent.TimeUnit;

3. 在android/app/src/main/java/com/{yourAppName}/MainApplication.java文件中添加：

```java
public void onCreate() {
      super.onCreate();
      Stetho.initializeWithDefaults(this);
      OkHttpClient client = new OkHttpClient.Builder()
      .connectTimeout(0, TimeUnit.MILLISECONDS)
      .readTimeout(0, TimeUnit.MILLISECONDS)
      .writeTimeout(0, TimeUnit.MILLISECONDS)
      .cookieJar(new ReactCookieJarContainer())
      .addNetworkInterceptor(new StethoInterceptor())
      .build();
      OkHttpClientProvider.replaceOkHttpClient(client);
}
```
4. 运行react-native run-android

5. 打开一个新的Chrome选项卡，在地址栏中输入chrome://inspect并回车。在页面中选择'Inspect device' （标有"Powered by Stetho"字样）。

### 调试原生代码

在和原生代码打交道时（比如编写原生模块），可以直接从Android Studio或是Xcode中启动应用，并利用这些IDE的内置功能来调试（比如设置断点）。这一方面和开发原生应用并无二致。

### 性能监测

你可以在开发者菜单中选择"Pref Monitor"选项以开启一个悬浮层，其中会显示应用的当前帧数。

