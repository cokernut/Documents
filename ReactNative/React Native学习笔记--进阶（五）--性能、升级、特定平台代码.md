---
title: React Native 学习笔记--进阶（五）--性能、升级版本、特定平台代码
date: 2016-12-09
tags: React Native
categories: React Native
description:
---
React Native 进阶（五）--性能、升级版本、特定平台代码
<!--more-->

## 性能

使用React Native替代基于WebView的框架来开发App的一个强有力的理由，就是为了使App可以达到每秒60帧（足够流畅），并且能有类似原生App的外观和手感。但是，还是有一些地方有所欠缺，以及在某些场合React Native还不能够替你决定如何进行优化，因此人工的干预依然是必要的。 

### 关于“帧”你所需要知道的

视频中逼真的动态效果其实是一种幻觉，这种幻觉是由一组静态的图片以一个稳定的速度快速变化所产生的。我们把这组图片中的每一张图片叫做一帧，而每秒钟显示的帧数直接的影响了视频（或者说用户界面）的流畅度和真实感。iOS设备提供了每秒60的帧率，这就留给了开发者和UI系统大约16.67ms来完成生成一张静态图片（帧）所需要的所有工作。如果在这分派的16.67ms之内没有能够完成这些工作，就会引发‘丢帧’的后果，使界面表现的不够流畅。

调出你应用的开发菜单，打开Show FPS Monitor. 你会注意到有两个不同的帧率（JS和UI）：

#### JavaScript 帧率

对大多数React Native应用来说，业务逻辑是运行在JavaScript线程上的。这是React应用所在的线程，也是发生API调用，以及处理触摸事件等操作的线程。更新数据到原生支持的视图是批量进行的，并且在事件循环每进行一次的时候被发送到原生端，这一步通常会在一帧时间结束之前处理完（一切顺利的话）。如果JavaScript线程有一帧没有及时响应，就被认为发生了一次丢帧。 例如：你在一个复杂应用的根组件上调用了this.setState，从而导致一次开销很大的子组件树的重绘，可想而知，这可能会花费200ms也就是整整12帧的丢失。此时，任何由JavaScript控制的动画都会卡住。只要卡顿超过100ms，用户就会明显的感觉到。

这种情况经常发生在Navigator的切换过程中：当你push一个新的路由时，JavaScript需要绘制新场景所需的所有组件，以发送正确的命令给原生端去创建视图。由于切换是由JavaScript线程所控制，因此经常会占用若干帧的时间，引起一些卡顿。有的时候，组件会在componentDidMount函数中做一些额外的事情，这甚至可能会导致页面切换过程中多达一秒的卡顿。

另一个例子是触摸事件的响应：如果你正在JavaScript线程处理一个跨越多个帧的工作，你可能会注意到TouchableOpacity的响应被延迟了。这是因为JavaScript线程太忙了，不能够处理主线程发送过来的原始触摸事件。结果TouchableOpacity就不能及时响应这些事件并命令主线程的页面去调整透明度了。

#### 主线程 (也即UI线程) 帧率

很多人会注意到，NavigatorIOS的性能要比Navigator好的多。原因就是它的切换动画是完全在主线程上执行的，因此不会被JavaScript线程上的掉帧所影响。

同样，当JavaScript线程卡住的时候，你仍然可以欢快的上下滚动ScrollView，因为ScrollView运行在主线程之上（尽管滚动事件会被分发到JS线程，但是接收这些事件对于滚动这个动作来说并不必要）。

### 性能问题的常见原因

#### console.log语句

在运行打好了离线包的应用时，控制台打印语句可能会极大地拖累JavaScript线程。注意有些第三方调试库也可能包含控制台打印语句，比如redux-logger，所以在发布应用前请务必仔细检查，确保全部移除。

> 有个babel插件可以帮你移除所有的console.*调用。首先需要使用npm install babel-plugin-transform-remove-console --save来安装，然后在项目根目录下编辑（或者是新建）一个名为·.babelrc`的文件，在其中加入：
```json
{
  "env": {
    "production": {
      "plugins": ["transform-remove-console"]
    }
  }
}
```
这样在打包发布时，所有的控制台语句就会被自动移除，而在调试时它们仍然会被正常调用。

#### 开发模式 (dev=true)

JavaScript线程的性能在开发模式下是很糟糕的。这是不可避免的，因为有许多工作需要在运行的时候去做，譬如使你获得良好的警告和错误信息，又比如验证属性类型（propTypes）以及产生各种其他的警告。

#### 缓慢的导航器(Navigator)切换

Navigator的动画是由JavaScript线程所控制的。想象一下“从右边推入”这个场景的切换：每一帧中，新的场景从右向左移动，从屏幕右边缘开始，最终移动到x轴偏移为0的屏幕位置。切换过程中的每一帧，JavaScript线程都需要发送一个新的x轴偏移量给主线程。如果JavaScript线程卡住了，它就无法处理这项事情，因而这一帧就无法更新，动画就被卡住了。

长远的解决方法，其中一部分是要允许基于JavaScript的动画从主线程分离。同样是上面的例子，我们可以在切换动画开始的时候计算出一个列表，其中包含所有的新的场景需要的x轴偏移量，然后一次发送到主线程以某种优化的方式执行。由于JavaScript线程已经从更新x轴偏移量给主线程这个职责中解脱了出来，因此JavaScript线程中的掉帧就不是什么大问题了 —— 用户将基本上不会意识到这个问题，因为用户的注意力会被流畅的切换动作所吸引。

不幸的是，这个方案还没有被实现。所以当前的解决方案是，在动画的进行过程中，利用InteractionManager来选择性的渲染新场景所需的最小限度的内容。

InteractionManager.runAfterInteractions的参数中包含一个回调，这个回调会在navigator切换动画结束的时候被触发（每个来自于Animated接口的动画都会通知InteractionManager）。

你的场景组件看上去应该是这样的：

```js
class ExpensiveScene extends React.Component {
  constructor(props, context) {
    super(props, context);
    this.state = {renderPlaceholderOnly: true};
  }

  componentDidMount() {
    InteractionManager.runAfterInteractions(() => {
      this.setState({renderPlaceholderOnly: false});
    });
  }

  render() {
    if (this.state.renderPlaceholderOnly) {
      return this._renderPlaceholderView();
    }

    return (
      <View>
        <Text>Your full view goes here</Text>
      </View>
    );
  }

  _renderPlaceholderView() {
    return (
      <View>
        <Text>Loading...</Text>
      </View>
    );
  }
};
```

你不必被限制在仅仅是做一些loading指示的渲染，你也可以绘制部分的页面内容 —— 例如：当你加载Facebook应用的时候，你会看见一个灰色方形的消息流的占位符，是将来用来显示文字的地方。如果你正在场景中绘制地图，那么最好在场景切换完成之前，显示一个灰色的占位页面或者是一个转动的动画，因为切换过程的确会导致主线程的掉帧。

#### ListView初始化渲染太慢以及列表过长时滚动性能太差

这是一个频繁出现的问题。因为iOS配备了UITableView，通过重用底层的UIViews实现了非常高性能的体验。用React Native实现相同效果的工作仍正在进行中，但是在此之前，我们有一些可用的方法来稍加改进性能以满足我们的需求。

##### initialListSize

这个属性定义了在首次渲染中绘制的行数。如果我们关注于快速的显示出页面，可以设置initialListSize为1，然后我们会发现其他行在接下来的帧中被快速绘制到屏幕上。而每帧所显示的行数由pageSize所决定。

##### pageSize

在初始渲染也就是initialListSize被使用之后，ListView将利用pageSize来决定每一帧所渲染的行数。默认值为1 —— 但是如果你的页面很小，而且渲染的开销不大的话，你会希望这个值更大一些。稍加调整，你会发现它所起到的作用。

##### scrollRenderAheadDistance

“在将要进入屏幕某些区域中先渲染行，距离按像素计算”

如果我们有一个2000个元素的列表，并且立刻全部渲染出来的话，无论是内存还是计算资源都会显得很匮乏。还很可能导致非常可怕的阻塞。因此scrollRenderAheadDistance允许我们来指定一个超过视野范围之外所需要渲染的行数。

##### removeClippedSubviews

“当这一选项设置为true的时候，超出屏幕的子视图（同时overflow值为hidden）会从它们原生的父视图中移除。这个属性可以在列表很长的时候提高滚动的性能。默认为true。（0.14版本前默认为false）”

这是一个应用在长列表上极其重要的优化。Android上，overflow值总是hidden的，所以你不必担心没有设置它。而在iOS上，你需要确保在行容器上设置了overflow: hidden。

#### 我的组件渲染太慢，我不需要立即显示全部

这在初次浏览ListView时很常见，适当的使用它是获得稳定性能的关键。就像之前所提到的，它可以提供一些手段在不同帧中来分开渲染页面，稍加改进就可以满足你的需求。此外要记住的是，ListView也可以横向滚动。

#### 在重绘一个几乎没有什么变化的页面时，JS帧率严重降低

如果你正在使用一个ListView，你必须提供一个rowHasChanged函数，它通过快速的算出某一行是否需要重绘，来减少很多不必要的工作。如果你使用了不可变的数据结构，这项工作就只需检查其引用是否相等。

同样的，你可以实现shouldComponentUpdate函数来指明在什么样的确切条件下，你希望这个组件得到重绘。如果你编写的是纯粹的组件（返回值完全由props和state所决定），你可以利用PureRenderMixin来为你做这个工作。再强调一次，不可变的数据结构在提速方面非常有用 —— 当你不得不对一个长列表对象做一个深度的比较，它会使重绘你的整个组件更加快速，而且代码量更少。

#### 由于在JavaScript线程中同时做很多事情，导致JS线程掉帧

“导航切换极慢”是该问题的常见表现。在其他情形下，这种问题也可能会出现。使用InteractionManager是一个好的方法，但是如果在动画中，为了用户体验的开销而延迟其他工作并不太能接受，那么你可以考虑一下使用LayoutAnimation。

Animated的接口一般会在JavaScript线程中计算出所需要的每一个关键帧，而LayoutAnimation则利用了Core Animation，使动画不会被JS线程和主线程的掉帧所影响。  
注意：LayoutAnimation只工作在“一次性”的动画上（"静态"动画） -- 如果动画可能会被中途取消，你还是需要使用Animated。

#### 在屏幕上移动视图（滚动，切换，旋转）时，UI线程掉帧

当具有透明背景的文本位于一张图片上时，或者在每帧重绘视图时需要用到透明合成的任何其他情况下，这种现象尤为明显。设置shouldRasterizeIOS或者renderToHardwareTextureAndroid属性可以显著改善这一现象。 注意不要过度使用该特性，否则你的内存使用量将会飞涨。在使用时，要评估你的性能和内存使用情况。如果你没有需要移动这个视图的需求，请关闭这一属性。

#### 使用动画改变图片的尺寸时，UI线程掉帧

在iOS上，每次调整Image组件的宽度或者高度，都需要重新裁剪和缩放原始图片。这个操作开销会非常大，尤其是大的图片。比起直接修改尺寸，更好的方案是使用transform: [{scale}]的样式属性来改变尺寸。比如当你点击一个图片，要将它放大到全屏的时候，就可以使用这个属性。

#### Touchable系列组件不能很好的响应

有些时候，如果我们有一项操作与点击事件所带来的透明度改变或者高亮效果发生在同一帧中，那么有可能在onPress函数结束之前我们都看不到这些效果。比如在onPress执行了一个setState的操作，这个操作需要大量计算工作并且导致了掉帧。对此的一个解决方案是将onPress处理函数中的操作封装到requestAnimationFrame中：

```js
handleOnPress() {
  // 谨记在使用requestAnimationFrame、setTimeout以及setInterval时
  // 要使用TimerMixin（其作用是在组件unmount时，清除所有定时器）
  this.requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```
#### 分析

你可以利用内置的分析器来同时获取JavaScript线程和主线程中代码执行情况的详细信息。

## 升级

时刻将React Native更新到最新的版本，可以获得更多API、视图、开发者工具以及其他一些好东西（官方开发任务繁重，人手紧缺，几乎不会对旧版本提供维护支持，所以即便更新可能带来一些兼容上的变更，但建议开发者还是尽一切可能第一时间更新）。由于一个完整的React Native项目是由Android项目、iOS项目和JavaScript项目组成的，且都打包在一个npm包中，所以升级可能会有一些麻烦。以下是目前所需的升级步骤：

### 更新react-native的node依赖包

打开项目目录下的package.json文件，然后在dependencies模块下找到react-native，将当前版本号改到最新（或指定）版本号，如：

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
react-native的npm包的最新版本可以去这里查看，或使用npm info react-native命令查看。

项目的根目录执行：

> npm install

安装最新的React Native版本,成功后可能会出现如下类似警告：

> npm WARN react-native@0.38.0 requires a peer of react@15.4.1 but none was installed.

根据警告执行：

> npm install –save react@15.4.1

更新最新的React且项目下package.json 的 dependencies下的react版本会被修改为 15.4.1

### 升级项目模板文件

新版本的npm包通常还会包含一些动态生成的文件，这些文件是在运行react-native init创建新项目时生成的，比如iOS和Android的项目文件。为了使老项目的项目文件也能得到更新（不重新init），你需要在命令行中运行：

> react-native upgrade

这一命令会检查最新的项目模板，然后进行如下操作：

+ 如果是新添加的文件，则直接创建。
+ 如果文件和当前版本的文件相同，则跳过。
+ 如果文件和当前版本的文件不同，则会提示你一些选项：查看两者的不同，选择保留你的版本或是用新的模板覆盖。你可以按下h键来查看所有可以使用的命令。

注意：如果你有修改原生代码，那么在使用upgrade升级前，先备份，再覆盖。覆盖完成后，使用比对工具找出差异，将你之前修改的代码逐步搬运到新文件中。

### 手动升级

有时候React Native的项目结构改动较大，此时还需要手动做一些修改，例如从0.13到0.14版本，或是0.28到0.29版本。所以在升级时请先阅读一下[更新日志](https://github.com/facebook/react-native/releases)，以确定是否需要做一些额外的手动修改。

### 查看版本是否升级成功

执行：

> react-native -v

通过如上命令来看最新的版本，检测是否升级成功！

## 特定平台代码

在制作跨平台的App时，多半会碰到针对不同平台编写不同代码的需求。最直接的方案就是把组件放置到不同的文件夹下：

> /common/components/   
> /android/components/   
> /ios/components/

另一个选择是根据平台不同在组件的文件命名上加以区分，如下：

> BigButtonIOS.js
> BigButtonAndroid.js

但除此以外React Native还提供了另外两种简单区分平台的方案：

### 特定平台扩展名

React Native会检测某个文件是否具有.ios.或是.android.的扩展名，然后根据当前运行的平台加载正确对应的文件。

假设你的项目中有如下两个文件：

> BigButton.ios.js
> BigButton.android.js

这样命名组件后你就可以在其他组件中直接引用，而无需关心当前运行的平台是哪个。

> import BigButton from './components/BigButton';

React Native会根据运行平台的不同引入正确对应的组件。

### 平台模块

React Native提供了一个检测当前运行平台的模块。如果组件只有一小部分代码需要依据平台定制，那么这个模块就可以派上用场。

```js
import { Platform, StyleSheet } from 'react-native';

var styles = StyleSheet.create({
  height: (Platform.OS === 'ios') ? 200 : 100,
});
```
Platform.OS在iOS上会返回ios，而在Android设备或模拟器上则会返回android。

还有个实用的方法是Platform.select()，它可以以Platform.OS为key，从传入的对象中返回对应平台的值，见下面的示例：

```js
import { Platform, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    ...Platform.select({
      ios: {
        backgroundColor: 'red',
      },
      android: {
        backgroundColor: 'blue',
      },
    }),
  },
});
```

上面的代码会根据平台的不同返回不同的container样式——iOS上背景色为红色，而android为蓝色。

这一方法可以接受任何合法类型的参数，因此你也可以直接用它针对不同平台返回不同的组件，像下面这样：

```js
const Component = Platform.select({
  ios: () => require('ComponentIOS'),
  android: () => require('ComponentAndroid'),
})();
<Component />;
```

#### 检测Android版本

在Android上，平台模块还可以用来检测当前所运行的Android平台的版本：

```js
import { Platform } from 'react-native';

if(Platform.Version === 21){
  console.log('Running on Lollipop!');
}
```