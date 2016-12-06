---
title: React Native 学习笔记--进阶（一）
date: 2016-12-03
tags: React Native
categories: React Native
description:
---
React Native 进阶（一）
<!--more-->

## 嵌入到Android原生应用中

参考：[React Native嵌入Android原生项目中](http://cokernut.top/2016/11/25/Android/React%20Native%E5%B5%8C%E5%85%A5Android%E5%8E%9F%E7%94%9F%E9%A1%B9%E7%9B%AE%E4%B8%AD/)

## 颜色

支持的颜色代码格式：

+ '#f0f' (#rgb)
+ '#f0fc' (#rgba)
+ '#ff00ff' (#rrggbb)
+ '#ff00ff00' (#rrggbbaa)
+ 'rgb(255, 255, 255)'
+ 'rgba(255, 255, 255, 1.0)'
+ 'hsl(360, 100%, 100%)'
+ 'hsla(360, 100%, 100%, 1.0)'
+ 'transparent'
+ 'red'
+ 0xff00ff00 (0xrrggbbaa)

## 图片

### 静态图片资源

React Native提供了一个统一的方式来管理iOS和Android应用中的图片。
你可以使用@2x，@3x这样的文件名后缀，来为不同的屏幕精度提供图片。比如下面这样的代码结构：

```
.
├── root.js
└── img
    ├── check@2x.png
    └── check@3x.png
```
并且root.js里有这样的代码：
```js
<Image source={require('./img/check.png')} />  
```
如果你有check@2x.ios.png和check@2x.android.png，Packager会打包所有的图片根据平台而选择不同的文件，并且依据屏幕精度提供对应的资源。
<font color="FA7F7F">注意：如果你添加图片的时候packager正在运行，可能需要重启packager以便能正确引入新添加的图片。<br>注意：为了使新的图片资源机制正常工作，require中的图片名字必须是一个静态字符串。</font>  

```js
// 正确
<Image source={require('./my-icon.png')} />
// 错误
var icon = this.props.active ? 'my-icon-active' : 'my-icon-inactive';
<Image source={require('./' + icon + '.png')} />
// 正确
var icon = this.props.active ? require('./my-icon-active.png') : require('./my-icon-inactive.png');
<Image source={icon} />
```

### 使用混合App的图片资源

如果你在编写一个混合App（一部分UI使用React Native，而另一部分使用平台原生代码），也可以使用已经打包到App中的图片资源（通过Xcode的asset类目或者Android的drawable文件夹打包）：
```js
<Image source={{uri: 'app_icon'}} style={{width: 40, height: 40}} />
```
<font color="FA7F7F">注意：这一做法并没有任何安全检查。你需要自己确保图片在应用中确实存在，而且还需要指定尺寸。</font>

### 网络图片

很多要在App中显示的图片<font color="FA7F7F">并不能在编译的时候获得，又或者有时候需要动态载入来减少打包后的二进制文件的大小。这些时候，与静态资源不同的是，你需要手动指定图片的尺寸。</font>
```js
// 正确
<Image source={{uri: 'https://facebook.github.io/react/img/logo_og.png'}}
       style={{width: 400, height: 400}} />
// 错误
<Image source={{uri: 'https://facebook.github.io/react/img/logo_og.png'}} />
```

### 本地文件系统中的图片

CameraRoll模块提供了访问本地相册的功能。

> static saveImageWithTag(tag) 

保存一个图片到相册。

@param {string} tag 
在安卓上，本参数是一个本地URI，例如"file:///sdcard/img.png".  
在iOS设备上可能是以下之一：
+ 本地URI
+ 资源库的标签
+ 非以上两种类型，表示图片数据将会存储在内存中（并且在本进程持续的时候一直会占用内存）。

返回一个Promise，操作成功时返回新的URI。

> static getPhotos(params: object) 

返回一个带有图片标识符对象的Promise。返回的对象的结构参见getPhotosReturnChecker:

```js
var getPhotosReturnChecker = createStrictShapeTypeChecker({
  edges: ReactPropTypes.arrayOf(createStrictShapeTypeChecker({
    node: createStrictShapeTypeChecker({
      type: ReactPropTypes.string.isRequired,
      group_name: ReactPropTypes.string.isRequired,
      image: createStrictShapeTypeChecker({
        uri: ReactPropTypes.string.isRequired,
        height: ReactPropTypes.number.isRequired,
        width: ReactPropTypes.number.isRequired,
        isStored: ReactPropTypes.bool,
      }).isRequired,
      timestamp: ReactPropTypes.number.isRequired,
      location: createStrictShapeTypeChecker({
        latitude: ReactPropTypes.number,
        longitude: ReactPropTypes.number,
        altitude: ReactPropTypes.number,
        heading: ReactPropTypes.number,
        speed: ReactPropTypes.number,
      }),
    }).isRequired,
  })).isRequired,
  page_info: createStrictShapeTypeChecker({
    has_next_page: ReactPropTypes.bool.isRequired,
    start_cursor: ReactPropTypes.string,
    end_cursor: ReactPropTypes.string,
  }).isRequired,
});
```

@param {object} 要求的参数结构参见getPhotosParamChecker：

```js
var getPhotosParamChecker = createStrictShapeTypeChecker({
  first: ReactPropTypes.number.isRequired,
  after: ReactPropTypes.string,
  groupTypes: ReactPropTypes.oneOf(GROUP_TYPES_OPTIONS),
  groupName: ReactPropTypes.string,
  assetType: ReactPropTypes.oneOf(ASSET_TYPE_OPTIONS),
  //例如：image/jpeg
  mimeTypes: ReactPropTypes.arrayOf(ReactPropTypes.string),
});
```

返回一个Promise，操作成功时返回符合getPhotosReturnChecker结构的对象。

#### 最合适的相册图片

iOS会为同一张图片在相册中保存多个不同尺寸的副本。为了性能考虑，从这些副本中挑出最合适的尺寸显得尤为重要。对于一处200x200大小的缩略图，显然不应该选择最高质量的3264x2448大小的图片。如果恰好有匹配的尺寸，那么React Native会自动为你选好。如果没有，则会选择最接近的尺寸进行缩放，但也至少缩放到比所需尺寸大出50%，以使图片看起来仍然足够清晰。这一切过程都是自动完成的，所以你不用操心自己去完成这些繁琐且易错的代码。

### 为什么不在所有情况下都自动指定尺寸

在浏览器中，如果你不给图片指定尺寸，那么浏览器会首先渲染一个0x0大小的元素占位，然后下载图片，在下载完成后再基于正确的尺寸来渲染图片。这样做的最大问题是UI会在图片加载的过程中上下跳动，使得用户体验非常糟糕。
在React Native中有意避免了这一行为。如此一来就需要做更多工作来提前知晓远程图片的尺寸（或宽高比），但我们相信这样可以带来更好的用户体验。然而，从已经打包好的应用资源文件中读取图片（使用require('image!x')语法）则无需指定尺寸，因为它们的尺寸在加载时就可以立刻知道。
这样一个引用require('image!logo')的实际输出结果可能是：
```
{"__packager_asset":true,"isStatic":true,"path":"/Users/react/HelloWorld/iOS/Images.xcassets/react.imageset/logo.png","uri":"logo","width":591,"height":573}
```

### 资源属性是一个对象（object）

在React Native中，把src属性改为了source属性，而且并不接受字符串，正确的值是一个带有uri属性的对象。
```
<Image source={{uri: 'something.jpg'}} />
```
这样可以使我们在对象中添加一些元数据(metadata)。假设你在使用require('./my-icon.png')，那么就会在其中添加真实文件路径以及尺寸等信息。对于开发者来说，则可以在其中标注一些有用的属性，例如图片的尺寸，这样可以使图片自己去计算将要显示的尺寸（而不必在样式中写死）。你可以在这一数据结构中自由发挥，存储你可能需要的任何图片相关的信息。

### 通过嵌套来实现背景图片

开发者们常面对的一种需求就是类似web中的背景图（background-image）。要实现这一用例，只需简单地创建一个<Image>组件，然后把需要背景图的子组件嵌入其中即可。

```js
return (
  <Image source={...}>
    <Text>Inside</Text>
  </Image>
);
```

## 触摸事件

React Native提供了可以处理常见触摸手势（例如点击或滑动）的组件， 以及可用于识别更复杂的手势的完整的手势响应系统。

### 可点击的组件

在需要捕捉用户点击操作时，可以使用"Touchable"开头的一系列组件。这些组件通过onPress属性接受一个点击事件的处理函数。当一个点击操作开始并且终止于本组件时（即在本组件上按下手指并且抬起手指时也没有移开到组件外），此函数会被调用。  
可点击的组件需要给用户提供视觉反馈，例如是哪个组件正在响应用户的操作，以及当用户抬起手指后会发生什么。用户也应该可以通过把手指移到一边来取消点击操作。  
具体使用哪种组件，取决于你希望给用户什么样的视觉反馈：
+ 一般来说，你可以使用TouchableHighlight来制作按钮或者链接。注意此组件的背景会在用户手指按下时变暗。  
+ 在Android上还可以使用TouchableNativeFeedback，它会在用户手指按下时形成类似墨水涟漪的视觉效果。  
+ TouchableOpacity会在用户手指按下时降低按钮的透明度，而不会改变背景的颜色。   
+ 如果你想在处理点击事件的同时不显示任何视觉反馈，则需要使用TouchableWithoutFeedback。

### 长按

某些场景中你可能需要检测用户是否进行了长按操作。可以在上面列出的任意组件中使用onLongPress属性来实现。

```js
import React, { Component } from 'react';
import {
  View,
  Image,
  Text,
  TouchableOpacity,
  StyleSheet
} from 'react-native';

class Advance extends Component {
  _click() {
    console.log("Click!");
  }

  _longClick() {
    console.log("Long Click!");
  }

  render() {
    return (
      <View>
        <TouchableOpacity onPress={this._click} onLongPress={this._longClick}>
          <Image source={{ uri: 'https://facebook.github.io/react/img/logo_og.png' }}
            style={[styles.image, { width: 200, height: 200 }]} >
            <Text style={styles.text}>背景图片</Text>
          </Image>
        </TouchableOpacity>
      </View>
    )
  }
}

const styles = StyleSheet.create({
  image: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center'
  },
  text: {
    fontSize: 40,
    textAlign: 'center',
    color: '#F67FF8',
  }
});

export default Advance;
```

### 在列表中上下滑动和在视图上左右滑动

可滚动的列表是移动应用中一个常见的模式。用户会在列表中或快或慢的各种滑动。ScrollView组件可以满足这一需求。
ScrollView可以在垂直或水平方向滚动，还可以配置pagingEnabled属性来让用户整屏整屏的滑动。此外，水平方向的滑动还可以使用Android上的ViewPagerAndroid 组件。
ListView则是一种特殊的ScrollView，用于显示比较长的垂直列表。它还可以显示分区块的头部和尾部，类似iOS上的UITableView控件。

#### 双指缩放

如果在ScrollView中只放置一个组件，则可以用来实现缩放操作。设置maximumZoomScale和minimumZoomScale属性即可以使用户能够缩放其中的内容

## 处理其他的手势

### 手势响应系统

移动设备上的手势识别要比在web上复杂得多。用户的一次触摸操作的真实意图是什么，App要经过好几个阶段才能判断。比如App需要判断用户的触摸到底是在滚动页面，还是滑动一个widget，或者只是一个单纯的点击。甚至随着持续时间的不同，这些操作还会转化。此外，还有多点同时触控的情况。

触摸响应系统可以使组件在不关心父组件或子组件的前提下自行处理触摸交互。

#### 最佳实践

用户之所以会觉得web app和原生app在体验上有巨大的差异，触摸响应是一大关键因素。用户的每一个操作都应该具有下列属性：

+ 反馈/高亮 —— 让用户看到他们到底按到了什么东西，以及松开手后会发生什么。
+ 取消功能 —— 当用户正在触摸操作时，应该是可以通过把手指移开来终止操作。  

这些特性使得用户在使用App时体验更好，因为它们可以让用户大胆试用，而不必担心点错了什么。

#### TouchableHighlight与Touchable系列组件

响应系统用起来可能比较复杂。所以官方提供了一个抽象的Touchable实现，用来做“可触控”的组件。这一实现利用了响应系统，使得我们可以简单地以声明的方式来配置触控处理。如果要做一个按钮或者网页链接，那么使用TouchableHighlight就可以。

#### 响应者的生命周期

一个View只要实现了正确的协商方法，就可以成为触摸事件的响应者。我们通过两个方法去“询问”一个View是否愿意成为响应者：

+ View.props.onStartShouldSetResponder: (evt) => true, - 在用户开始触摸的时候（手指刚刚接触屏幕的瞬间），是否愿意成为响应者？
+ View.props.onMoveShouldSetResponder: (evt) => true, - 如果View不是响应者，那么在每一个触摸点开始移动（没有停下也没有离开屏幕）时再询问一次：是否愿意响应触摸交互呢？  

如果View返回true，并开始尝试成为响应者，那么会触发下列事件之一:

+ View.props.onResponderGrant: (evt) => {} - View现在要开始响应触摸事件了。这也是需要做高亮的时候，使用户知道他到底点到了哪里。
+ View.props.onResponderReject: (evt) => {} - 响应者现在“另有其人”而且暂时不会“放权”，请另作安排。

如果View已经开始响应触摸事件了，那么下列这些处理函数会被一一调用：

+ View.props.onResponderMove: (evt) => {} - 用户正在屏幕上移动手指时（没有停下也没有离开屏幕）。
+ View.props.onResponderRelease: (evt) => {} - 触摸操作结束时触发，比如"touchUp"（手指抬起离开屏幕）。
+ View.props.onResponderTerminationRequest: (evt) => true - 有其他组件请求接替响应者，当前的View是否“放权”？返回true的话则释放响应者权力。
+ View.props.onResponderTerminate: (evt) => {} - 响应者权力已经交出。这可能是由于其他View通过onResponderTerminationRequest请求的，也可能是由操作系统强制夺权（比如iOS上的控制中心或是通知中心）。

evt是一个合成事件，它包含以下结构：  
+ nativeEvent
    + changedTouches - 在上一次事件之后，所有发生变化的触摸事件的数组集合（即上一次事件后，所有移动过的触摸点）
    + identifier - 触摸点的ID
    + locationX - 触摸点相对于父元素的横坐标
    + locationY - 触摸点相对于父元素的纵坐标
    + pageX - 触摸点相对于根元素的横坐标
    + pageY - 触摸点相对于根元素的纵坐标
    + target - 触摸点所在的元素ID
    + timestamp - 触摸事件的时间戳，可用于移动速度的计算
    + touches - 当前屏幕上的所有触摸点的集合

#### 捕获ShouldSet事件处理
onStartShouldSetResponder与onMoveShouldSetResponder是以冒泡的形式调用的，即嵌套最深的节点最先调用。这意味着当多个View同时在*ShouldSetResponder中返回true时，最底层的View将优先“夺权”。在多数情况下这并没有什么问题，因为这样可以确保所有控件和按钮是可用的。

但是有些时候，某个父View会希望能先成为响应者。我们可以利用“捕获期”来解决这一需求。响应系统在从最底层的组件开始冒泡之前，会首先执行一个“捕获期”，在此期间会触发on*ShouldSetResponderCapture系列事件。因此，如果某个父View想要在触摸操作开始时阻止子组件成为响应者，那就应该处理onStartShouldSetResponderCapture事件并返回true值。

> View.props.onStartShouldSetResponderCapture: (evt) => true,
> View.props.onMoveShouldSetResponderCapture: (evt) => true,

### PanResponder

PanResponder类可以将多点触摸操作协调成一个手势。它使得一个单点触摸可以接受更多的触摸操作，也可以用于识别简单的多点触摸手势。

它提供了一个对触摸响应系统响应器的可预测的包装。对于每一个处理函数，它在原生事件之外提供了一个新的gestureState对象。

> onPanResponderMove: (event, gestureState) => {}

原生事件是指由以下字段组成的合成触摸事件：

+ nativeEvent
    + changedTouches - 在上一次事件之后，所有发生变化的触摸事件的数组集合（即上一次事件后，所有移动过的触摸点）
    + identifier - 触摸点的ID
    + locationX - 触摸点相对于父元素的横坐标
    + locationY - 触摸点相对于父元素的纵坐标
    + pageX - 触摸点相对于根元素的横坐标
    + pageY - 触摸点相对于根元素的纵坐标
    + target - 触摸点所在的元素ID
    + timestamp - 触摸事件的时间戳，可用于移动速度的计算
    + touches - 当前屏幕上的所有触摸点的集合

一个gestureState对象有如下的字段：

+ stateID - 触摸状态的ID。在屏幕上有至少一个触摸点的情况下，这个ID会一直有效。
+ moveX - 最近一次移动时的屏幕横坐标
+ moveY - 最近一次移动时的屏幕纵坐标
+ x0 - 当响应器产生时的屏幕坐标
+ y0 - 当响应器产生时的屏幕坐标
+ dx - 从触摸操作开始时的累计横向路程
+ dy - 从触摸操作开始时的累计纵向路程
+ vx - 当前的横向移动速度
+ vy - 当前的纵向移动速度
+ numberActiveTouches - 当前在屏幕上的有效触摸点的数量

基本用法

```js
componentWillMount: function() {
    this._panResponder = PanResponder.create({
        // 要求成为响应者：
        onStartShouldSetPanResponder: (evt, gestureState) => true,
        onStartShouldSetPanResponderCapture: (evt, gestureState) => true,
        onMoveShouldSetPanResponder: (evt, gestureState) => true,
        onMoveShouldSetPanResponderCapture: (evt, gestureState) => true,

        onPanResponderGrant: (evt, gestureState) => {
        // 开始手势操作。给用户一些视觉反馈，让他们知道发生了什么事情！
        // gestureState.{x,y}0 现在会被设置为0
        },
        onPanResponderMove: (evt, gestureState) => {
        // 最近一次的移动距离为gestureState.move{X,Y}
        // 从成为响应者开始时的累计手势移动距离为gestureState.d{x,y}
        },
        onPanResponderTerminationRequest: (evt, gestureState) => true,
        onPanResponderRelease: (evt, gestureState) => {
        // 用户放开了所有的触摸点，且此时视图已经成为了响应者。
        // 一般来说这意味着一个手势操作已经成功完成。
        },
        onPanResponderTerminate: (evt, gestureState) => {
        // 另一个组件已经成为了新的响应者，所以当前手势将被取消。
        },
        onShouldBlockNativeResponder: (evt, gestureState) => {
        // 返回一个布尔值，决定当前组件是否应该阻止原生组件成为JS响应者
        // 默认返回true。目前暂时只支持android。
        return true;
        },
    });
},

render: function() {
    return (
        <View {...this._panResponder.panHandlers} />
    );
},
```

#### 方法

> static create(config: object) 

@param {object} 配置所有响应器回调的加强版本，不仅仅包括原本的ResponderSyntheticEvent，还包括PanResponder手势状态的回调。你只要简单的把onResponder回调中的Responder替换为PanResponder。举例来说，这个config对象可能看起来像这样：

+ onMoveShouldSetPanResponder: (e, gestureState) => {...}
+ onMoveShouldSetPanResponderCapture: (e, gestureState) => {...}
+ onStartShouldSetPanResponder: (e, gestureState) => {...}
+ onStartShouldSetPanResponderCapture: (e, gestureState) => {...}
+ onPanResponderReject: (e, gestureState) => {...}
+ onPanResponderGrant: (e, gestureState) => {...}
+ onPanResponderStart: (e, gestureState) => {...}
+ onPanResponderEnd: (e, gestureState) => {...}
+ onPanResponderRelease: (e, gestureState) => {...}
+ onPanResponderMove: (e, gestureState) => {...}
+ onPanResponderTerminate: (e, gestureState) => {...}
+ onPanResponderTerminationRequest: (e, gestureState) => {...}
+ onShouldBlockNativeResponder: (e, gestureState) => {...}

通常来说，对那些有对应捕获事件的事件来说，我们在捕获阶段更新gestureState一次，然后在冒泡阶段直接使用即可。

注意onStartShould 回调。他们只会在此节点冒泡/捕获的开始/结束事件中提供已经更新过的gestureState。一旦这个节点成为了事件的响应者，则所有的开始/结束事件都会被手势正确处理，并且gestureState也会被正确更新。(numberActiveTouches)有可能没有包含所有的触摸点，除非你就是触摸事件的响应者。

