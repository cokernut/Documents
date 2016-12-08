---
title: React Native 学习笔记--基础知识
date: 2016-12-01
tags: React Native
categories: React Native
description:
---
React Native 基础知识学习
<!--more-->

## Hello World!

```javascript
import React, { Component } from 'react';
import { AppRegistry, Text } from 'react-native';

class HelloWorld extends Component {
  render() {
    return (
      <Text>Hello world!</Text>
    );
  }
}

/**  
 * 注册应用(registerComponent)后才能正确渲染
 * 注意：一般在整个应用里AppRegistry.registerComponent这个方法只会调用一次，
 * 而不是每个组件/模块都注册
 * 注意，这里用引号括起来的'HelloWorldApp'必须和你init创建的项目名一致
 */
AppRegistry.registerComponent('HelloWorldApp', () => HelloWorld);
```

## Props(属性)、State(状态)、Style(样式)

大多数组件在创建时就可以使用各种参数来进行定制。用于定制的这些参数就称为props（属性）。

以常见的基础组件Image为例，在创建一个图片时，可以传入一个名为source的prop来指定要显示的图片的地址，
以及使用名为style的prop来控制其尺寸。

自定义的组件也可以使用props。通过在不同的场景使用不同的属性定制，可以尽量提高自定义组件的复用范畴。
只需在render函数中引用this.props，然后按需处理即可。

```javascript
import React, { Component } from 'react';
import {
  AppRegistry,
  StyleSheet,
  Text,
  Image,
  View
} from 'react-native';

class MyTextView extends Component {
  constructor(props) {
    super(props);
    this.state = {showText: true}; //初始化state（状态）
    // 每1000毫秒对showText状态做一次取反操作
    setInterval(() => {
        this.setState({showText: !this.state.showText}) //调用setState方法修改state的值
    }, 10000);
  }
  render() {
    let temp = 'Name: ' + this.props.name + '  Age: ' + this.props.age + '!'; //取属性值并拼装
    let text = this.state.showText ? temp : ''; 
    return (
      <Text style={styles.text}>{text}</Text>
    );
  }
}

export default class Root extends Component {
  render() {
    let pic={
      uri:'http://file26.mafengwo.net/M00/25/15/wKgB4lIre0yAC1WOAAFKo9uhzX063.rbook_comment.w300.jpeg'
    }

    return (
      /**
       * props,state,style
       * 
       * style属性可以是一个普通的JavaScript对象,
       * 你可以传入一个数组——在数组中位置居后的样式对象比居前的优先级更高，这样你可以间接实现样式的继承。
       * 常见的做法是按顺序声明和使用style属性，以借鉴CSS中的“层叠”做法（即后声明的属性会覆盖先声明的同名属性）。
       * 
       * 传值到Image的source属性，并定义样式
       * 属性传值，this.props.propsname调用
       */
      <View>
        <Text style={[styles.text, styles.gray, {fontSize: 30}]}>Hello World!</Text>
        <Image source={pic} style={{width:100, height:100}}></Image>
        <MyTextView name='Tom' age='16'/> 
        <MyTextView name='Joy' age='18'/>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  text: {
    color: 'green',
    fontSize: 20,
    fontWeight: 'bold'
  },
  gray: {
    color: 'gray'
  }
});

AppRegistry.registerComponent('ReactNativeDemo', () => Root);
```

## 布局

### 指定宽高：

最简单的给组件设定尺寸的方式就是在样式中指定固定的width和height。 React Native中的尺寸都是无单位的，表示的是与设备像素密度无关的逻辑像素点。

### 弹性（Flex）宽高：

在组件样式中使用flex可以使其在可利用的空间中动态地扩张或收缩。一般而言我们会使用flex:1来指定某个组件扩张以撑满所有剩余的空间。
如果有多个并列的子组件使用了flex:1，则这些子组件会平分父容器中剩余的空间。如果这些并列的子组件的flex值不一样，则谁的值更大，
谁占据剩余空间的比例就更大（即占据剩余空间的比等于并列组件间flex值的比）。  
<font color="#D96060">注意：  
组件能够撑满剩余空间的前提是其父容器的尺寸不为零。如果父容器既没有固定的width和height，
也没有设定flex，则父容器的尺寸为零。其子组件如果使用了flex，也是无法显示的。</font>

```javascript
import React, { Component } from 'react';
import { AppRegistry, View } from 'react-native';

export default class Root extends Component {
  render() {
    return (
      // 试试去掉父View中的`flex: 1`。
      // 则父View不再具有尺寸，因此子组件也无法再撑开。
      // 然后再用`height: 300`来代替父View的`flex: 1`试试看？
      <View style={{flex: 1}}>
        <View style={{flex: 1, backgroundColor: 'powderblue'}} />
        <View style={{flex: 2, backgroundColor: 'skyblue'}} />
        <View style={{flex: 3, backgroundColor: 'steelblue'}} />
      </View>
    );
  }
}

AppRegistry.registerComponent('ReactNativeDemo', () => Root);
```

## Flexbox布局

### flexDirection: 

决定布局的主轴，默认值是column(竖直轴，起点在顶部)，而不是row(水平轴，起点在左端)

### alignItems: 

交叉轴的对齐方式，默认值是stretch  
- flex-start: 交叉轴的起点对齐
- flex-end: 交叉轴的终点对齐
- center: 交叉轴的中心对齐
- stretch: 容器中的所有项目拉伸填满整个容器  

### alignSelf:

当前组件交叉轴的对齐方式，会覆盖的父组件的alignItems属性

### justifyContent:   

可以决定其子组件沿着主轴的对齐方式，默认值是flex-start  
- flex-start：主轴起点对齐
- flex-end：主轴终点
- center：居中
- space-between：两端对齐，项目之间的间隔都相等
- space-around: 每个项目两侧的间隔相等。项目之间的间隔比项目与边框的间隔大一倍。

### flexWrap:   

flexWrap属性定义一条轴线排不下时是否折行。它有两个值，分别是’wrap’和’nowrap’，分别代表支持换行和不支持换行，默认是’nowrap’。

### flex:   
只能指定一个数字值，这个属性可能使属性justifyContent失效。

### 属性列表  

属性                            |       取值
------                          |--------
alignItems    交叉轴的对齐方式   | enum('flex-start', 'flex-end', 'center', 'stretch') 
alignSelf 组件交叉轴的对齐方式   | enum('auto', 'flex-start', 'flex-end', 'center', 'stretch') 
borderBottomWidth 下边框宽度    | number 
borderLeftWidth   左边框宽度    | number 
borderRightWidth  右边框宽度    | number 
borderTopWidth    上边框宽度    | number 
borderWidth        边框宽度     | number 
bottom         覆盖下边缘宽度    | number 
flex              权重          | number 
flexDirection     主轴          | enum('row', 'column') 
flexWrap          是否折行      | enum('wrap', 'nowrap') 
height            高度          | number 
justifyContent 主轴的排列方式    | enum('flex-start', 'flex-end', 'center', 'space-between', 'space-around') 
left           覆盖左边缘宽度    | number 
margin         组件外边距        | number 
marginBottom   组件下外边距      | number 
marginHorizontal 组件横向外边距  | number 
marginLeft     组件左外边距      | number 
marginRight    组件右外边距      | number 
marginTop       组件上外边距     | number 
marginVertical  组件竖向外边距   | number 
padding        组件内边距        | number 
paddingBottom  组件下内边距      | number 
paddingHorizontal 组件横向内边距 | number 
paddingLeft    组件左内边距      | number 
paddingRight   组件右内边距      | number 
paddingTop     组件上内边距      | number 
paddingVertical 组件竖向内边距   | number 
position 位置属性 默认relative   | enum('absolute', 'relative')(绝对位置，相对位置) 
right       覆盖右边缘宽度       | number 
top         覆盖上边缘宽度       | number 
width           宽度            | number 
zIndex   元素的堆叠顺序（z轴）    | number

## View

View是一个支持Flexbox布局、样式、一些触摸处理、和一些无障碍功能的容器，并且它可以放到其它的视图里，也可以有任意多个任意类型的子视图。

### 属性

> accessibilityLabel string 

设置当用户与此元素交互时，“读屏器”（对视力障碍人士的辅助功能）阅读的文字。默认情况下，这个文字会通过遍历所有的子元素并累加所有的文本标签来构建。

> accessible bool 

当此属性为true时，表示此视图时一个启用了无障碍功能的元素。默认情况下，所有可触摸操作的元素都是无障碍功能元素。

> onAccessibilityTap function 

当accessible为true时，如果用户对一个已选中的无障碍元素做了一个双击手势时，系统会调用此函数。（译注：此事件是针对残障人士，并非是一个普通的点击事件。如果要为View添加普通点击事件，请直接使用Touchable系列组件替代View，然后添加onPress函数）。

> onLayout function 

当组件挂载或者布局变化的时候调用，参数为：  
{nativeEvent: { layout: {x, y, width, height}}}  
这个事件会在布局计算完成后立即调用一次，不过收到此事件时新的布局可能还没有在屏幕上呈现，尤其是一个布局动画正在进行中的时候。

> onMagicTap function 

当accessible为true时，如果用户做了一个双指轻触(Magic tap)手势，系统会调用此函数。

> onResponderGrant function 

对于大部分的触摸处理，你只需要用TouchableHighlight或TouchableOpacity包装你的组件。

> pointerEvents enum('box-none', 'none', 'box-only', 'auto') 

用于控制当前视图是否可以作为触控事件的目标。  
auto：视图可以作为触控事件的目标。  
none：视图不能作为触控事件的目标。  
box-none：视图自身不能作为触控事件的目标，但其子视图可以。  
box-only：视图自身可以作为触控事件的目标，但其子视图不能。  

> removeClippedSubviews bool 

这是一个特殊的性能相关的属性，由RCTView导出。在制作滑动控件时，如果控件有很多不在屏幕内的子视图，会非常有用。

要让此属性生效，首先要求视图有很多超出范围的子视图，并且子视图和容器视图（或它的某个祖先视图）都应该有样式overflow: hidden。

> style
```
Flexbox...
ShadowProp#style...
Transforms...
backfaceVisibility enum('visible', 'hidden')
backgroundColor string
borderColor string
borderTopColor string
borderRightColor string
borderBottomColor string
borderLeftColor string
borderRadius number
borderTopLeftRadius number
borderTopRightRadius number
borderBottomLeftRadius number
borderBottomRightRadius number
borderStyle enum('solid', 'dotted', 'dashed')
borderWidth number
borderTopWidth number
borderRightWidth number
borderBottomWidth number
borderLeftWidth number
opacity number
overflow enum('visible', 'hidden')
elevation number
(限Android)使用Android原生的 elevation API来设置视图的高度（elevation）。
这样可以为视图添加一个投影，并且会影响视图层叠的顺序。此属性仅支持Android5.0及以上版本。
```

## Text

一个用于显示文本的React组件，并且它也支持嵌套、样式，以及触摸处理。
在React Native中你必须把你的文本节点放在<Text>组件内。你不能直接在<View>下放置一段文本。

### 属性

> numberOfLines number 

用来当文本过长的时候裁剪文本。包括折叠产生的换行在内，总的行数不会超过这个属性的限制。

> onLayout function 

当挂载或者布局变化以后调用，参数为如下的内容：

{nativeEvent: {layout: {x, y, width, height}}}

> onLongPress function 

当文本被长按以后调用此回调函数。

> onPress function 

当文本被点击以后调用此回调函数。

> selectable function 

决定用户是否可以长按选择文本，以便复制和粘贴。

> style style 
```
View#style...
color string
fontFamily string
fontSize number
fontStyle enum('normal', 'italic')
fontWeight enum("normal", 'bold', '100', '200', '300', '400', '500', '600', '700', '800', '900')
指定字体的粗细。大多数字体都支持'normal'和'bold'值。并非所有字体都支持所有的数字值。如果某个值不支持，则会自动选择最接近的值。
letterSpacing number
lineHeight number
textAlign enum("auto", 'left', 'right', 'center', 'justify')
指定文本的对齐方式。其中'justify'值仅iOS支持。
android textAlignVertical enum('auto', 'top', 'bottom', 'center')
ios letterSpacing number
ios textDecorationColor string
textDecorationLine enum("none", 'underline', 'line-through', 'underline line-through')
ios textDecorationStyle enum("solid", 'double', 'dotted', 'dashed')
ios writingDirection enum("auto", 'ltr', 'rtl')
```
> testID string 

用来在端到端测试中标记这个视图。



## TextInput

TextInput是一个允许用户输入文本的基础组件。它有一个名为onChangeText的属性，此属性接受一个函数，而此函数会在文本变化时被调用。
另外还有一个名为onSubmitEditing的属性，会在文本被提交后（用户按下软键盘上的提交键）调用。

```javascript
class Root extends Component {
  constructor(props) {
    super(props);
    this.state = {text: ''};
  }

  render() {
    return (
      <View style={{padding: 10}}>
        <TextInput
          style={{height: 40}}
          placeholder="Type here to translate!"
          // 如果key和value的字面一样，那么可以简写成一个，等同于下面的写法：
          // this.setState({text: text});
          onChangeText={(text) => this.setState({text})}
        />
        <Text style={{padding: 10, fontSize: 42}}>
          {this.state.text.split(' ').map((word) => word && '🍕').join(' ')}
        </Text>
      </View>
    );
  }
}
```

### 属性

> autoCapitalize enum('none', 'sentences', 'words', 'characters') 

控制TextInput是否要自动将特定字符切换为大写：

characters: 所有的字符。
words: 每个单词的第一个字符。
sentences: 每句话的第一个字符（默认）。
none: 不自动切换任何字符为大写。  

> autoCorrect bool 

如果为false，会关闭拼写自动修正。默认值是true。

> autoFocus bool 

如果为true，在componentDidMount后会获得焦点。默认值为false。

> blurOnSubmit bool 

如果为true，文本框会在提交的时候失焦。对于单行输入框默认值为true，多行则为false。注意：对于多行输入框来说，如果将blurOnSubmit设为true，则在按下回车键时就会失去焦点同时触发onSubmitEditing事件，而不会换行。

> defaultValue string 

提供一个文本框中的初始值。当用户开始输入的时候，值就可以改变。

在一些简单的使用情形下，如果你不想用监听消息然后更新value属性的方法来保持属性和状态同步的时候，就可以用defaultValue来代替。

> editable bool 

如果为false，文本框是不可编辑的。默认值为true。

> keyboardType enum("default", 'numeric', 'email-address', "ascii-capable", 'numbers-and-punctuation', 'url', 'number-pad', 'phone-pad', 'name-phone-pad', 'decimal-pad', 'twitter', 'web-search') 

决定弹出的何种软键盘的，譬如numeric（纯数字键盘）。

这些值在所有平台都可用：

default
numeric
email-address

> maxLength number 

限制文本框中最多的字符数。使用这个属性而不用JS逻辑去实现，可以避免闪烁的现象。

> multiline bool 

如果为true，文本框中可以输入多行文字。默认值为false。

> onBlur function 

当文本框失去焦点的时候调用此回调函数。

> onChange function 

当文本框内容变化时调用此回调函数。

> onChangeText function 

当文本框内容变化时调用此回调函数。改变后的文字内容会作为参数传递。

> onEndEditing function 

当文本输入结束后调用此回调函数。

> onFocus function 

当文本框获得焦点的时候调用此回调函数。

> onLayout function 

当组件挂载或者布局变化的时候调用，参数为{x, y, width, height}。

> onSubmitEditing function 

此回调函数当软键盘的确定/提交按钮被按下的时候调用此函数。如果multiline={true}，此属性不可用。

> placeholder string 

如果没有任何文字输入，会显示此字符串。

> placeholderTextColor string 

占位字符串显示的文字颜色。

> secureTextEntry bool 

如果为true，文本框会遮住之前输入的文字，这样类似密码之类的敏感文字可以更加安全。默认值为false。

> selectTextOnFocus bool 

如果为true，当获得焦点的时候，所有的文字都会被选中。

> selectionColor string 

设置输入框高亮时的颜色（在iOS上还包括光标）

> style Text#style 

译注：这意味着本组件继承了所有Text的样式。

> value string 

文本框中的文字内容。

TextInput是一个受约束的(Controlled)的组件，意味着如果提供了value属性，原生值会被强制与value属性保持一致。在大部分情况下这都工作的很好，不过有些情况下会导致一些闪烁现象——一个常见的原因就是通过不改变value来阻止用户进行编辑。如果你希望阻止用户输入，可以考虑设置editable={false}；如果你是希望限制输入的长度，可以考虑设置maxLength属性，这两个属性都不会导致闪烁。

> ios clearButtonMode enum('never', 'while-editing', 'unless-editing', 'always') 

是否要在文本框右侧显示“清除”按钮。

> ios clearTextOnFocus bool 

如果为true，每次开始输入的时候都会清除文本框的内容。

> ios enablesReturnKeyAutomatically bool 

如果为true，键盘会在文本框内没有文字的时候禁用确认按钮。默认值为false。

> ios keyboardAppearance enum('default', 'light', 'dark') 

指定键盘的颜色。

> ios onKeyPress function 

当一个键被按下的时候调用此回调。被按下的键会作为参数传递给回调函数。会在onChange之前调用。

> ios returnKeyType enum('default', 'go', 'google', 'join', 'next', 'route', 'search', 'send', 'yahoo', 'done', 'emergency-call')

决定“确定”按钮显示的内容。

> ios selectionState DocumentSelectionState 

参见DocumentSelectionState.js，可以控制一个文档中哪段文字被选中的状态。

> android numberOfLines number 

设置输入框的行数。当multiline设置为true时使用它，可以占据对应的行数。

> android underlineColorAndroid string 

文本框的下划线颜色(译注：如果要去掉文本框的边框，请将此属性设为透明transparent)。


### 方法

> isFocused(): boolean #

返回值表明当前输入框是否获得了焦点。

> clear() 

清空输入框的内容。

## ScrollView

ScrollView是一个通用的可滚动的容器，你可以在其中放入多个组件和视图，而且这些组件并不需要是同类型的。
ScrollView不仅可以垂直滚动，还能水平滚动（通过horizontal属性来设置）。  
ScrollView适合用来显示数量不多的滚动元素。

### 属性

> contentContainerStyle StyleSheetPropType(ViewStylePropTypes) 

这些样式会应用到一个内层的内容容器上，所有的子视图都会包裹在内容容器内。例子：

```javascript
return (
    <ScrollView contentContainerStyle={styles.contentContainer}>
    </ScrollView>
  );
  ...
  const styles = StyleSheet.create({
    contentContainer: {
      paddingVertical: 20
    }
  });
```

> horizontal bool 

当此属性为true的时候，所有的的子视图会在水平方向上排成一行，而不是默认的在垂直方向上排成一列。默认值为false。

> keyboardDismissMode enum('none', "interactive", 'on-drag') 

用户拖拽滚动视图的时候，是否要隐藏软键盘。

none（默认值），拖拽时不隐藏软键盘。

on-drag 当拖拽开始的时候隐藏软键盘。

interactive 软键盘伴随拖拽操作同步地消失，并且如果往上滑动会恢复键盘。安卓设备上不支持这个选项，会表现的和none一样。

> keyboardShouldPersistTaps bool 

当此属性为false的时候，在软键盘激活之后，点击焦点文本输入框以外的地方，键盘就会隐藏。如果为true，滚动视图不会响应点击操作，并且键盘不会自动消失。默认值为false。

> onContentSizeChange function 

此函数会在ScrollView内部可滚动内容的视图发生变化时调用。

调用参数为内容视图的宽和高: (contentWidth, contentHeight)

此方法是通过绑定在内容容器上的onLayout来实现的。

> onScroll function 

在滚动的过程中，每帧最多调用一次此回调函数。调用的频率可以用scrollEventThrottle属性来控制。

> refreshControl element 

指定RefreshControl组件，用于为ScrollView提供下拉刷新功能。

> removeClippedSubviews bool #

（实验特性）：当此属性为true时，屏幕之外的子视图（子视图的overflow样式需要设为hidden）会被移除。这个可以提升大列表的滚动性能。默认值为true。

> showsHorizontalScrollIndicator bool 

当此属性为true的时候，显示一个水平方向的滚动条。

> showsVerticalScrollIndicator bool 

当此属性为true的时候，显示一个垂直方向的滚动条。

> pagingEnabled bool 

当值为true时，滚动条会停在滚动视图的尺寸的整数倍位置。这个可以用在水平分页上。默认值为false。

> scrollEnabled bool 

当值为false的时候，内容不能滚动，默认值为true。

> style属性
```
Flexbox...
ShadowProp#style...
Transforms...
backfaceVisibility enum('visible', 'hidden')
backgroundColor string
borderColor string
borderTopColor string
borderRightColor string
borderBottomColor string
borderLeftColor string
borderRadius number
borderTopLeftRadius number
borderTopRightRadius number
borderBottomLeftRadius number
borderBottomRightRadius number
borderStyle enum('solid', 'dotted', 'dashed')
borderWidth number
borderTopWidth number
borderRightWidth number
borderBottomWidth number
borderLeftWidth number
opacity number
overflow enum('visible', 'hidden')
shadowColor string
shadowOffset {width: number, height: number}
shadowOpacity number
shadowRadius number
```
### 方法
> scrollTo(y: number | { x?: number, y?: number, animated?: boolean }, x: number, animated: boolean) 

滚动到指定的x, y偏移处。第三个参数为是否启用平滑滚动动画。  
使用示例:  
scrollTo({x: 0, y: 0, animated: true})

## ListView

ListView组件用于显示一个垂直的滚动列表，其中的元素之间结构近似而仅数据不同。  
ListView更适于长列表数据，且元素个数可以增删。和ScrollView不同的是，ListView并不立即渲染所有元素，而是优先渲染屏幕上可见的元素。  
ListView最基本的使用方式就是创建一个ListView.DataSource数据源，然后给它传递一个普通的数据数组，
再使用数据源来实例化一个ListView组件，并且定义它的renderRow回调函数，这个函数会接受数组中的每个数据作为参数，返回一个可渲染的组件（作为listview的每一行）。
rowHasChanged函数也是ListView的必需属性。(===符号只比较基本类型数据的值，和引用类型的地址)

```javascript
constructor(props) {
  super(props);
  var ds = new ListView.DataSource({rowHasChanged: (r1, r2) => r1 !== r2});
  this.state = {
    dataSource: ds.cloneWithRows(['row 1', 'row 2']),
  };
}
render() {
  return (
    <ListView
      dataSource={this.state.dataSource}
      renderRow={(rowData) => <Text>{rowData}</Text>}
    />
  );
}
```

ListView还支持一些高级特性，譬如给每段/组(section)数据添加一个带有粘性的头部（类似iPhone的通讯录，其首字母会在滑动过程中吸附在屏幕上方）；
在列表头部和尾部增加单独的内容；在到达列表尾部的时候调用回调函数(onEndReached)，还有在视野内可见的数据变化时调用回调函数(onChangeVisibleRows)，
以及一些性能方面的优化。  
有一些性能优化使得ListView可以滚动的更加平滑，尤其是在动态加载可能很大（或者概念上无限长的）数据集的时候：  
+ 只更新变化的行 - 提供给数据源的rowHasChanged函数可以告诉ListView它是否需要重绘一行数据（即：数据是否发生了变化）参见ListViewDataSource
+ 限制频率的行渲染 - 默认情况下，每次消息循环只有一行会被渲染（可以用pageSize属性配置）。这把较大的工作分散成小的碎片，以降低因为渲染而导致丢帧的可能性。

### 属性

> ScrollView props... 

ListView可以使用所有ScrollView的属性。

> dataSource ListViewDataSource 

ListView.DataSource实例（列表依赖的数据源）

> initialListSize number 

指定在组件刚挂载的时候渲染多少行数据。用这个属性来确保首屏显示合适数量的数据，而不是花费太多帧逐步显示出来。

> onChangeVisibleRows function 

(visibleRows, changedRows) => void

当可见的行的集合变化的时候调用此回调函数。visibleRows 以 { sectionID: { rowID: true }}的格式包含了所有可见行，而changedRows 以{ sectionID: { rowID: true | false }}的格式包含了所有刚刚改变了可见性的行，其中如果值为true表示一个行变得可见，而为false表示行刚刚离开可视区域而变得不可见。

> onEndReached function 

当所有的数据都已经渲染过，并且列表被滚动到距离最底部不足onEndReachedThreshold个像素的距离时调用。原生的滚动事件会被作为参数传递。译注：当第一次渲染时，如果数据不足一屏（比如初始值是空的），这个事件也会被触发，请自行做标记过滤。

> onEndReachedThreshold number 

调用onEndReached之前的临界值，单位是像素。

> pageSize number 

每次事件循环（每帧）渲染的行数。

> removeClippedSubviews bool 

用于提升大列表的滚动性能。需要给行容器添加样式overflow:'hidden'。（Android已默认添加此样式）。此属性默认开启。

> renderFooter function 

() => renderable

页头与页脚会在每次渲染过程中都重新渲染（如果提供了这些属性）。如果它们重绘的性能开销很大，把他们包装到一个StaticContainer或者其它恰当的结构中。页脚会永远在列表的最底部，而页头会在最顶部。

> renderHeader function 

> renderRow function 

(rowData, sectionID, rowID, highlightRow) => renderable

从数据源(Data source)中接受一条数据，以及它和它所在section的ID。返回一个可渲染的组件来为这行数据进行渲染。默认情况下参数中的数据就是放进数据源中的数据本身，不过也可以提供一些转换器。

如果某一行正在被高亮（通过调用highlightRow函数），ListView会得到相应的通知。当一行被高亮时，其两侧的分割线会被隐藏。行的高亮状态可以通过调用highlightRow(null)来重置。

> renderScrollComponent function 

(props) => renderable

指定一个函数，在其中返回一个可以滚动的组件。ListView将会在该组件内部进行渲染。默认情况下会返回一个包含指定属性的ScrollView。

> renderSectionHeader function 

(sectionData, sectionID) => renderable

如果提供了此函数，会为每个小节(section)渲染一个粘性的标题。

粘性是指当它刚出现时，会处在对应小节的内容顶部；继续下滑当它到达屏幕顶端的时候，它会停留在屏幕顶端，一直到对应的位置被下一个小节的标题占据为止。

> renderSeparator function 

(sectionID, rowID, adjacentRowHighlighted) => renderable

如果提供了此属性，一个可渲染的组件会被渲染在每一行下面，除了小节标题的前面的最后一行。在其上方的小节ID和行ID，以及邻近的行是否被高亮会作为参数传递进来。

> scrollRenderAheadDistance number 

当一个行接近屏幕范围多少像素之内的时候，就开始渲染这一行。

### 方法
> getMetrics() 

导出一些用于性能分析的数据。

> scrollTo(...args) 

滚动到指定的x, y偏移处，可以指定是否加上过渡动画。
参考:ScrollView#scrollTo.

## 网络

### 使用Fetch

React Native提供了和web标准一致的Fetch API，用于满足开发者访问网络的需求。

> 发起网络请求

要从任意地址获取内容的话，只需简单地将网址作为参数传递给fetch方法即可（fetch这个词本身也就是获取的意思）：

```javascript
fetch('https://mywebsite.com/mydata.json')
```

Fetch还有可选的第二个参数，可以用来定制HTTP请求一些参数。你可以指定header参数，或是指定使用POST方法，又或是提交数据等等：

```javascript
fetch('https://mywebsite.com/endpoint/', {
  method: 'POST',
  headers: {
    'Accept': 'application/json',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    firstParam: 'yourValue',
    secondParam: 'yourOtherValue',
  })
})
```
如果你的服务器无法识别上面POST的数据格式，那么可以尝试传统的form格式，示例如下：

```javascript
fetch('https://mywebsite.com/endpoint/', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
  },
  body: 'key1=value1&key2=value2'
})
```
可以参考Fetch请求文档来查看所有可用的参数。

> 处理服务器的响应数据

网络请求天然是一种异步操作。Fetch 方法会返回一个Promise，这种模式可以简化异步风格的代码：

```javascript
getMoviesFromApiAsync() {
  return fetch('http://facebook.github.io/react-native/movies.json')
    .then((response) => response.json())
    .then((responseJson) => {
      return responseJson.movies;
    })
    .catch((error) => {
      console.error(error);
    });
}
```
你也可以在React Native应用中使用ES7标准中的async/await 语法：

```javascript
// 注意这个方法前面有async关键字
async getMoviesFromApi() {
  try {
    // 注意这里的await语句，其所在的函数必须有async关键字声明
    let response = await fetch('http://facebook.github.io/react-native/movies.json');
    let responseJson = await response.json();
    return responseJson.movies;
  } catch(error) {
    console.error(error);
  }
}
```
别忘了catch住fetch可能抛出的异常，否则出错时你可能看不到任何提示。

### 使用其他的网络库

React Native中已经内置了XMLHttpRequest API(也就是俗称的ajax)。一些基于XMLHttpRequest封装的第三方库也可以使用，
例如frisbee或是axios等。但注意不能使用jQuery，因为jQuery中还使用了很多浏览器中才有而RN中没有的东西（所以也不是所有web中的ajax库都可以直接使用）。

### WebSocket

React Native还支持WebSocket，这种协议可以在单个TCP连接上提供全双工的通信信道。
WebSocket一开始的握手需要借助HTTP请求完成，可以实现实时通讯。

```javascript
var ws = new WebSocket('ws://host.com/path');

ws.onopen = () => {
  // 打开一个连接
  ws.send('something'); // 发送一个消息
};

ws.onmessage = (e) => {
  // 接收到了一个消息
  console.log(e.data);
};

ws.onerror = (e) => {
  // 发生了一个错误
  console.log(e.message);
};

ws.onclose = (e) => {
  // 连接被关闭了
  console.log(e.code, e.reason);
};
```

# 导航器

## Navigator

### 场景（Scene）的概念与使用

无论是View中包含Text，还是一个排满了图片的ScrollView，渲染各种组件现在对你来说应该已经得心应手了。
这些摆放在一个屏幕中的组件，就共同构成了一个“场景（Scene）”。场景简单来说其实就是一个全屏的React组件。

下面定义了一个仅显示一些文本的简单场景：

> MyScene.js：

```javascript
import React, { Component } from 'react';
import { View, Text } from 'react-native';

export default class MyScene extends Component {
  static defaultProps = {
    title: 'MyScene'
  };

  render() {
    return (
      <View>
        <Text>Hi! My name is {this.props.title}.</Text>
      </View>
    )
  }
}
```
<font color="#D96060">
注意:组件声明前面的export default关键字。它的意思是导出(export)当前组件，以允许其他组件引入(import)和使用当前组件。
</font>

> index.android.js

```javascript
import React, { Component } from 'react';
import { AppRegistry } from 'react-native';

// ./MyScene表示的是当前目录下的MyScene.js文件
// 注意即便当前文件和MyScene.js在同一个目录中，"./"两个符号也是不能省略的！
// 但是.js后缀是可以省略的

import MyScene from './MyScene';

class YoDawgApp extends Component {
  render() {
    return (
      <MyScene />
    )
  }
}

AppRegistry.registerComponent('YoDawgApp', () => YoDawgApp);
```
我们现在已经创建了只有单个场景的App。其中的MyScene同时也是一个可复用的Reac组件的例子。

### 使用Navigator

下面我们开始尝试导航跳转。首先要做的是渲染一个Navigator组件，然后通过此组件的renderScene属性方法来渲染其他场景。

```javascript
render() {
  return (
    <Navigator
      initialRoute={{ title: 'My Initial Scene', index: 0 }}
      renderScene={(route, navigator) => {
        return <MyScene title={route.title} />
      }}
    />
  );
}
```
使用导航器经常会碰到“路由(route)”的概念。“路由”抽象自现实生活中的路牌，在RN中专指包含了场景信息的对象。
renderScene方法是完全根据路由提供的信息来渲染场景的，你可以在路由中任意自定义参数以区分标记不同的场景。

#### 将场景推入导航栈

要过渡到新的场景，你需要了解push和pop方法。这两个方法由navigator对象提供，而这个对象就是上面的renderScene方法中传递的第二个参数。 
我们使用这两个方法来把路由对象推入或弹出导航栈。

```javascript
navigator.push({
  title: 'Next Scene',
  index: 1,
});

navigator.pop();
```
```javascript
import React, { Component } from 'react';
import { AppRegistry, Navigator, Text, View } from 'react-native';

import MyScene from './MyScene';

class SimpleNavigationApp extends Component {
  render() {
    return (
      <Navigator
        initialRoute={{ title: 'My Initial Scene', index: 0 }}
        renderScene={(route, navigator) =>
          <MyScene
            title={route.title}

            // 调用这个函数显示一个新的场景           
            onForward={ () => {    
              const nextIndex = route.index + 1;
              navigator.push({
                title: 'Scene ' + nextIndex,
                index: nextIndex,
              });
            }}

            // 调用这个函数回到当前场景
            onBack={() => {
              if (route.index > 0) {
                navigator.pop();
              }
            }}
          />
        }
      />
    )
  }
}

AppRegistry.registerComponent('SimpleNavigationApp', () => SimpleNavigationApp);
```

> MyScene.js

```javascript
import React, { Component, PropTypes } from 'react';
import { View, Text, TouchableHighlight } from 'react-native';

export default class MyScene extends Component {
  static propTypes = {
    title: PropTypes.string.isRequired,
    onForward: PropTypes.func.isRequired,
    onBack: PropTypes.func.isRequired,
  }
  render() {
    return (
      <View>
        <Text>Current Scene: { this.props.title }</Text>
        <TouchableHighlight onPress={this.props.onForward}>
          <Text>点我进入下一场景</Text>
        </TouchableHighlight>
        <TouchableHighlight onPress={this.props.onBack}>
          <Text>点我返回上一场景</Text>
        </TouchableHighlight>    
      </View>
    )
  }
}
```
MyScene通过title属性接受了路由对象中的title值。它还包含了两个可点击的组件TouchableHighlight，
会在点击时分别调用通过props传入的onForward和onBack方法，而这两个方法各自调用了navigator.push()和navigator.pop()，从而实现了场景的变化。

