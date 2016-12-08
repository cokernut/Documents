---
title: React Native 学习笔记--进阶（二）--动画
date: 2016-12-06
tags: React Native
categories: React Native
description:
---
React Native 进阶（二）--动画
<!--more-->

## 动画

流畅、有意义的动画对于移动应用用户体验来说是非常必要的。我们可以联合使用两个互补的系统：用于全局的布局动画LayoutAnimation，和用于创建更精细的交互控制的动画Animated。

### Animated

Animated库使得开发者可以非常容易地实现各种各样的动画和交互方式，并且具备极高的性能。Animated仅关注动画的输入与输出声明，在其中建立一个可配置的变化函数，然后使用简单的start/stop方法来控制动画按顺序执行。  
示例：

```js
import React, { Component } from 'react';
import {
  Animated,
} from 'react-native';

class Simple extends Component {
  _callBack() {
      console.log("call Back");
  }
  constructor(props) {
    super(props);
    this.state = {
      bounceValue: new Animated.Value(0),
    };
  }
  render() {
    return (
      <Animated.Image                         // 可选的基本组件类型: Image, Text, View
        source={{uri: 'http://i.imgur.com/XMKOH81.jpg'}}
        style={{
          flex: 1,
          transform: [                        // `transform`是一个有序数组（动画按顺序执行）
            {scale: this.state.bounceValue},  // 将`bounceValue`赋值给 `scale`比例
          ]
        }}
      />
    );
  }
  componentDidMount() {
    this.state.bounceValue.setValue(1.5);     // 设置一个较大的初始值
    Animated.spring(                          // 可选的基本动画类型: spring, decay, timing
      this.state.bounceValue,                 // 将`bounceValue`值动画化
      {
        toValue: 0.8,                         // 将其值以动画的形式改到一个较小值
        friction: 2,                          // 弹性系数（摩擦力），用于回弹，数值越大弹性越小
      }
    ).start(this._callBack);                  // 开始执行动画，并传入回调函数
  }
}
export default Simple;
```
bounceValue在构造函数中初始化为state的一部分，然后和图片的缩放比例进行绑定。在动画执行的背后，其数值会被不断的计算并用于设置缩放比例。当组件刚刚挂载的时候，缩放比例被设置到1.5。然后紧跟着在bounceValue上执行了一个弹跳动画(spring)，会逐帧刷新数值，并同步更新所有依赖本数值的绑定（在这个例子里，就是图片的缩放比例）。比起调用setState然后重新渲染，这一运行过程要快得多。因为整个配置都是声明式的，我们可以实现更进一步的优化，只要序列化好配置，然后我们可以在一个高优先级的线程执行动画。

#### 核心API

大部分你需要的东西都来自Animated模块。它包括两个值类型，Value用于单个的值，而ValueXY用于向量值；还包括三种动画类型，spring，decay，还有timing，以及三种组件类型，View，Text和Image。你可以使用Animated.createAnimatedComponent方法来对其它类型的组件创建动画。

这三种动画类型可以用来创建几乎任何你需要的动画曲线，因为它们每一个都可以被自定义：

+ spring: 基础的单次弹跳物理模型，符合Origami设计标准
  + friction: 摩擦力，默认为7.
  + tension: 张力，默认40。

+ decay: 以一个初始速度开始并且逐渐减慢停止。
  + velocity: 起始速度，必填参数。
  + deceleration: 速度衰减比例，默认为0.997。

+ timing: 从时间范围映射到渐变的值。
  + duration: 动画持续的时间（单位是毫秒），默认为500。
  + easing：一个用于定义曲线的渐变函数。阅读Easing模块可以找到许多预定义的函数。iOS默认为Easing.inOut(Easing.ease)。
  + delay: 在一段时间之后开始动画（单位是毫秒），默认为0。
  
动画可以通过调用start方法来开始。start接受一个回调函数，当动画结束的时候会调用此回调函数。如果动画是因为正常播放完成而结束的，回调函数被调用时的参数为{finished: true}，但若动画是在结束之前被调用了stop而结束（可能是被一个手势或者其它的动画打断），它会收到参数{finished: false}。

#### 组合动画

多个动画可以通过parallel（同时执行）、sequence（顺序执行）、stagger（错开执行，可能会同时（不同步）在执行，延迟时间不一样，其实就是插入了delay的parrllel）和delay（延期执行）来组合使用。它们中的每一个都接受一个要执行的动画数组，并且自动在适当的时候调用start/stop。举个例子：

```js
Animated.sequence([            // 首先执行decay动画，结束后同时执行spring和twirl动画
  Animated.decay(position, {   // 滑行一段距离后停止
    velocity: {x: gestureState.vx, y: gestureState.vy}, // 根据用户的手势设置速度
    deceleration: 0.997,
  }),
  Animated.parallel([          // 在decay之后并行执行：
    Animated.spring(position, {
      toValue: {x: 0, y: 0}    // 返回到起始点开始
    }),
    Animated.timing(twirl, {   // 同时开始旋转
      toValue: 360,
    }),
  ]),
]).start();                    // 执行这一整套动画序列
```
默认情况下，如果任何一个动画被停止或中断了，组内所有其它的动画也会被停止。Parallel有一个stopTogether属性，如果设置为false，可以禁用自动停止。

#### 插值（interpolate）

Animated API还有一个很强大的部分就是interpolate插值函数。它可以接受一个输入区间，然后将其映射到另一个的输出区间。下面是一个一个简单的从0-1区间到0-100区间的映射示例：

```js
value.interpolate({
  inputRange: [0, 1],
  outputRange: [0, 100],
});
```
interpolate还支持定义多个区间段落，常用来定义静止区间等。举个例子，要让输入在接近-300时取相反值，然后在输入接近-100时到达0，然后在输入接近0时又回到1，接着一直到输入到100的过程中逐步回到0，最后形成一个始终为0的静止区间，对于任何大于100的输入都返回0。具体写法如下：

```js
value.interpolate({
  inputRange: [-300, -100, 0, 100, 101],
  outputRange: [300,    0, 1,   0,   0],
});
```
它的最终映射结果如下：

输入 |-400 | -300 | -200 | -100 | -50 | 0 | 50 | 100 | 101 | 200  
-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----
输出 |450  | 300  | 150  |  0  | 0.5  | 1 | 0.5 | 0  |  0  |  0

interpolate还支持到字符串的映射，从而可以实现颜色以及带有单位的值的动画变换。例如你可以像下面这样实现一个旋转动画：

```js
 value.interpolate({
   inputRange: [0, 360],
   outputRange: ['0deg', '360deg']
 })
```
interpolation还支持任意的渐变函数，其中有很多已经在Easing类中定义了，包括二次、指数、贝塞尔等曲线以及step、bounce等方法。interpolation还支持限制输出区间outputRange。你可以通过设置extrapolate、extrapolateLeft或extrapolateRight属性来限制输出区间。默认值是extend（允许超出），不过你可以使用clamp选项来阻止输出值超过outputRange。

#### 跟踪动态值

动画中所设的值还可以通过跟踪别的值得到。你只要把toValue设置成另一个动态值而不是一个普通数字就行了。比如我们可以用弹跳动画来实现聊天头像的闪动，又比如通过timing设置duration:0来实现快速的跟随。他们还可以使用插值来进行组合：

```js
Animated.spring(follower, {toValue: leader}).start();
Animated.timing(opacity, {
  toValue: pan.x.interpolate({
    inputRange: [0, 300],
    outputRange: [1, 0],
  }),
}).start();
```

ValueXY是一个方便的处理2D交互的办法，譬如旋转或拖拽。它是一个简单的包含了两个Animated.Value实例的包装，然后提供了一系列辅助函数，使得ValueXY在许多时候可以替代Value来使用。比如在上面的代码片段中，leader和follower可以同时为valueXY类型，这样x和y的值都会被跟踪。

#### 输入事件

Animated.event是Animated API中与输入有关的部分，允许手势或其它事件直接绑定到动态值上。它通过一个结构化的映射语法来完成，使得复杂事件对象中的值可以被正确的解开。第一层是一个数组，允许同时映射多个值，然后数组的每一个元素是一个嵌套的对象。在下面的例子里，你可以发现scrollX被映射到了event.nativeEvent.contentOffset.x(event通常是回调函数的第一个参数)，并且pan.x和pan.y分别映射到gestureState.dx和gestureState.dy（gestureState是传递给PanResponder回调函数的第二个参数）。

```
onScroll={Animated.event(
  [{nativeEvent: {contentOffset: {x: scrollX}}}]   // scrollX = e.nativeEvent.contentOffset.x
)}
onPanResponderMove={Animated.event([
  null,                                          // 忽略原生事件
  {dx: pan.x, dy: pan.y}                         // 从gestureState中解析出dx和dy的值
]);
```

#### 响应当前的动画值

你可能会注意到这里没有一个明显的方法来在动画的过程中读取当前的值——这是出于优化的角度考虑，有些值只有在原生代码运行阶段中才知道。如果你需要在JavaScript中响应当前的值，有两种可能的办法：

  + spring.stopAnimation(callback)会停止动画并且把最终的值作为参数传递给回调函数callback——这在处理手势动画的时候非常有用。
  + spring.addListener(callback) 会在动画的执行过程中持续异步调用callback回调函数，提供一个最近的值作为参数。这在用于触发状态切换的时候非常有用，譬如当用户拖拽一个东西靠近的时候弹出一个新的气泡选项。不过这个状态切换可能并不会十分灵敏，因为它不像许多连续手势操作（如旋转）那样在60fps下运行。

<font color="#F77F7F">注意</font>只有声明为可动画化的组件才能被关联动画。View、Text，还有Image都是可动画化的。如果你想让自定义组件可动画化，可以用createAnimatedComponent。这些特殊的组件里面用了一些黑魔法，来把动画数值绑定到属性上，然后在每帧去执行原生更新，来避免每次render和同步过程的开销。他们还处理了在节点卸载时的清理工作以确保使用安全。

动画具备很强的可配置性。自定义或者预定义的过渡函数、延迟、时间、衰减比例、刚度等等。取决于动画类型的不同，你还可以配置更多的参数。

一个Animated.Value可以驱动任意数量的属性，并且每个属性可以配置一个不同的插值函数。插值函数把一个输入的范围映射到输出的范围，通常我们用线性插值，不过你也可以使用其他的过渡函数。默认情况下，当输入超出范围时，它也会对应的进行转换，不过你也可以把输出约束到范围之内。

举个例子，你可能希望你的Animated.Value从0变化到1时，把组件的位置从150px移动到0px，不透明度从0到1。可以通过以下的方法修改style属性来实现：

```js
style={{
  opacity: this.state.fadeAnim, // Binds directly
  transform: [{
    translateY: this.state.fadeAnim.interpolate({
      inputRange: [0, 1],
      outputRange: [150, 0]  // 0 : 150, 0.5 : 75, 1 : 0
    }),
  }],
}}>
```

动画还可以被更复杂地组合，通过一些辅助函数例如sequence或者parallel（它们分别用于先后执行多个动画和同时执行多个动画），而且还可以通过把toValue设置为另一个Animated.Value来产生一个动画序列。

Animated.ValueXY则用来处理一些2D动画，譬如滑动。并且还有一些辅助功能譬如setOffset和getLayout来帮助实现一些常见的交互效果，譬如拖放操作(Drag and drop)。

注意Animated模块被设计为可完全序列化的，这样动画可以脱离JavaScript事件循环，以一种高性能的方式运行。这可能会导致API看起来比较难懂，与一个完全同步的动画系统相比稍微有一些奇怪。Animated.Value.addListener可以帮助你解决一些相关限制，不过使用它的时候需要小心，因为将来的版本中它可能会牵扯到性能问题。

#### 方法

> static decay(value: AnimatedValue | AnimatedValueXY, config: DecayAnimationConfig) 

推动一个值以一个初始的速度和一个衰减系数逐渐变为0。

> static timing(value: AnimatedValue | AnimatedValueXY, config: TimingAnimationConfig) 

推动一个值按照一个过渡曲线而随时间变化。Easing模块定义了一大堆曲线，你也可以使用你自己的函数。

> static spring(value: AnimatedValue | AnimatedValueXY, config: SpringAnimationConfig) 

产生一个基于Rebound和Origami实现的Spring动画。它会在toValue值更新的同时跟踪当前的速度状态，以确保动画连贯。可以链式调用。

> static add(a: Animated, b: Animated) 

将两个动画值相加计算，创建一个新的动画值。

> static multiply(a: Animated, b: Animated) 

将两个动画值相乘计算，创建一个新的动画值。

> static delay(time: number) 

在指定的延迟之后开始动画。

> static sequence(animations: Array<CompositeAnimation>) 

按顺序执行一个动画数组里的动画，等待一个完成后再执行下一个。如果当前的动画被中止，后面的动画则不会继续执行。

> static parallel(animations: Array<CompositeAnimation>, config?: ParallelConfig) 

同时开始一个动画数组里的全部动画。默认情况下，如果有任何一个动画停止了，其余的也会被停止。你可以通过stopTogether选项来改变这个效果。

> static stagger(time: number, animations: Array<CompositeAnimation>) 

一个动画数组，里面的动画有可能会同时执行（重叠），不过会以指定的延迟来开始。用来制作拖尾效果非常合适。

> static event(argMapping: Array<Mapping>, config?: EventConfig) 

接受一个映射的数组，对应的解开每个值，然后调用所有对应的输出的setValue方法。例如：

```
 onScroll={this.AnimatedEvent(
   [{nativeEvent: {contentOffset: {x: this._scrollX}}}]
   {listener},          // 可选的异步监听函数
 )
 ...
 onPanResponderMove: this.AnimatedEvent([
   null,                // 忽略原始事件
   {dx: this._panX},    // 手势状态参数
 ]),
```

> static createAnimatedComponent(Component: any) 

使得任何一个React组件支持动画。用它来创建Animated.View等等。

#### 属性
> Value: AnimatedValue 

表示一个数值的类，用于驱动动画。通常用new Animated.Value(0);来初始化。

> ValueXY: AnimatedValueXY 

表示一个2D值的类，用来驱动2D动画，例如拖动操作等。

#### class AnimatedValue

用于驱动动画的标准值。一个Animated.Value可以用一种同步的方式驱动多个属性，但同时只能被一个行为所驱动。启用一个新的行为（譬如开始一个新的动画，或者运行setValue）会停止任何之前的动作。

##### 方法
> constructor(value: number) 

> setValue(value: number) 

直接设置它的值。这个会停止任何正在进行的动画，然后更新所有绑定的属性。

> setOffset(offset: number) 

设置一个相对值，不论接下来的值是由setValue、一个动画，还是Animated.event产生的，都会加上这个值。常用来在拖动操作一开始的时候用来记录一个修正值（譬如当前手指位置和View位置）。

> flattenOffset() 

把当前的相对值合并到值里，并且将相对值设为0。最终输出的值不会变化。常在拖动操作结束后调用。

> addListener(callback: ValueListenerCallback) 

添加一个异步监听函数，这样你就可以监听动画值的变更。这有时候很有用，因为你没办法同步的读取动画的当前值，因为有时候动画会在原生层次运行。

> removeListener(id: string) 

> removeAllListeners() 

> stopAnimation(callback?: ?(value: number) => void) 

停止任何正在运行的动画或跟踪值。callback会被调用，参数是动画结束后的最终值，这个值可能会用于同步更新状态与动画位置。

> interpolate(config: InterpolationConfigType) 

在更新属性之前对值进行插值。譬如：把0-1映射到0-10。

> animate(animation: Animation, callback: EndCallback) 

一般仅供内部使用。不过有可能一个自定义的动画类会用到此方法。

> stopTracking() 

仅供内部使用。

> track(tracking: Animated) 

仅供内部使用。

#### class AnimatedValueXY
用来驱动2D动画的2D值，譬如滑动操作等。API和普通的Animated.Value几乎一样，只不过是个复合结构。它实际上包含两个普通的Animated.Value。

例子：

```js
class DraggableView extends React.Component {
   constructor(props) {
     super(props);
     this.state = {
       pan: new Animated.ValueXY(), // inits to zero
     };
     this.state.panResponder = PanResponder.create({
       onStartShouldSetPanResponder: () => true,
       onPanResponderMove: Animated.event([null, {
         dx: this.state.pan.x, // x,y are Animated.Value
         dy: this.state.pan.y,
       }]),
       onPanResponderRelease: () => {
         Animated.spring(
           this.state.pan,         // Auto-multiplexed
           {toValue: {x: 0, y: 0}} // Back to zero
         ).start();
       },
     });
   }
   render() {
     return (
       <Animated.View
         {...this.state.panResponder.panHandlers}
         style={this.state.pan.getLayout()}>
         {this.props.children}
       </Animated.View>
     );
   }
 }
```
##### 方法
> constructor(valueIn?: ?{x: number | AnimatedValue; y: number | AnimatedValue}) 

> setValue(value: {x: number; y: number}) 

> setOffset(offset: {x: number; y: number}) 

> flattenOffset() 

> stopAnimation(callback?: ?() => number) 

> addListener(callback: ValueXYListenerCallback) 

> removeListener(id: string) 

> getLayout() 

将一个{x, y}组合转换为{left, top}以用于样式。例如：

```
 style={this.state.anim.getLayout()}
```
> getTranslateTransform() 

将一个{x, y} 组合转换为一个可用的位移变换(translation transform)，例如：
```
 style={{
   transform: this.state.anim.getTranslateTransform()
 }}
```

综合示例：

```js
import React, { Component } from 'react';
import {
  Animated,
  View,
  Text,
  Easing
} from 'react-native';
/**
 * 弹性动画 ,基础的单次弹跳物理模型
class Simple extends Component {
  _callBack() {
      console.log("call Back");
  }
  constructor(props) {
    super(props);
    this.state = {
      bounceValue: new Animated.Value(0),
    };
  }
  render() {
    return (
      <Animated.Image                         // 可选的基本组件类型: Image, Text, View
        source={{uri: 'http://i.imgur.com/XMKOH81.jpg'}}
        style={{
          flex: 1,
          transform: [                        // `transform`是一个有序数组（动画按顺序执行）
            {scale: this.state.bounceValue},  // 将`bounceValue`赋值给 `scale`比例属性
          ]
        }}
      />
    );
  }
  componentDidMount() {
    this.state.bounceValue.setValue(1.5);     // 设置一个较大的初始值
    Animated.spring(                          // 可选的基本动画类型: spring, decay, timing
      this.state.bounceValue,                 // 将`bounceValue`值动画化
      {
        toValue: 0.8,                         // 将其值以动画的形式改到一个较小值
        friction: 2,                          // 弹性系数（摩擦力），用于回弹，数值越大弹性越小，默认7
        tension: 10                           // 张力，默认40。
      }
    ).start(this._callBack);                  // 开始执行动画，并传入回调函数
  }
}
*/
/**
 * 透明度位移动画,以一个初始速度开始并且逐渐减慢停止。

class Simple extends Component {
  constructor(props) {
    super(props);
    this.state = {
      anim: new Animated.Value(0.5),
    };
  }
  componentDidMount() {
    Animated.decay(
      this.state.anim,
      { 
        toValue: 2,
        velocity: 0.2, //起始速度，必填参数。
        deceleration: 0.99, //速度衰减比例，默认为0.997。
      },
    ).start();
  }
  render() {
    return (
      <Animated.Image
        source={require('../img/img.jpg')}
        style={{ //把值赋给透明度属性,Animated.Value从0变化到1时，把组件的位置从150px移动到0px，不透明度从0到1
          opacity: this.state.anim, // Binds directly
          transform: [
            {translateY: this.state.anim.interpolate({ //Y轴位移变化
              inputRange: [0, 20],
              outputRange: [0, 400]
            })}
          ],
        }}>
      </Animated.Image>
    );
  }
}
 */
class Simple extends Component {
  constructor(props) {
    super(props);
    this.state = {
      anim: new Animated.Value(0), //初始化，value=0
    }
  }

  componentDidMount() {
    Animated.sequence([      //顺序执行
      Animated.timing(      //从时间范围映射到渐变的值
        this.state.anim,
        {
          toValue: 1, //value要从0变为1
          easing: Easing.linear, //变化方式
          duration: 2500, //动画时间，默认500
          delay: 1000 //在一段时间之后开始动画（单位是毫秒），默认为0。
        }
      ),
      Animated.delay(2000), //延迟执行
    ]).start(this._callBack);
  }

  _callBack() {
    console.log("call Back");
  }

  render() {
    return(
      <Animated.Image
        source={require('../img/img.jpg')}
        style={{
          opacity: this.state.anim, //透明度变化
          transform: [
            {scale: this.state.anim.interpolate({ //缩放变化
              inputRange: [0, 1],
              outputRange: [1, 0.5],
            })},
            {translateX: this.state.anim.interpolate({ //X轴位移变化
              inputRange: [0, 1],
              outputRange: [0, 150]
            })},
            {translateY: this.state.anim.interpolate({ //Y轴位移变化
              inputRange: [0, 1],
              outputRange: [0, 150]
            })},
            {translateY: this.state.anim.interpolate({ //Y轴位移变化
              inputRange: [0, 1],
              outputRange: [0, 150]
            })},
            {rotateZ: this.state.anim.interpolate({ //Z轴旋转变化
              inputRange: [0, 1],
              outputRange: ['0deg', '360deg']
            })},
            {rotateX: this.state.anim.interpolate({ //X轴旋转变化
              inputRange: [0, 1],
              outputRange: ['0deg', '360deg']
            })},
            {rotateY: this.state.anim.interpolate({ //Y轴旋转变化
              inputRange: [0, 1],
              outputRange: ['0deg', '360deg']
            })},
          ]
        }}
      />
    );
  }
}
export default Simple;
```

### LayoutAnimation

LayoutAnimation的作用是当布局变化时，自动将视图运动到它们新的位置上，LayoutAnimation允许你在全局范围内创建和更新动画，这些动画会在下一次渲染或布局周期运行。它常用来更新flexbox布局，因为它可以无需测量或者计算特定属性就能直接产生动画。尤其是当布局变化可能影响到父节点（譬如“查看更多”展开动画既增加父节点的尺寸又会将位于本行之下的所有行向下推动）时，如果不使用LayoutAnimation，可能就需要显式声明组件的坐标，才能使得所有受影响的组件能够同步运行动画。

注意尽管LayoutAnimation非常强大且有用，但它对动画本身的控制没有Animated或者其它动画库那样方便，所以如果你使用LayoutAnimation无法实现一个效果，那可能还是要考虑其他的方案。

另外，如果要在Android上使用LayoutAnimation，那么目前还需要在UIManager中启用：

> UIManager.setLayoutAnimationEnabledExperimental && UIManager.setLayoutAnimationEnabledExperimental(true);

一个常用的调用此API的办法是调用LayoutAnimation.configureNext，然后调用setState。

#### 方法

> static configureNext(config: Config, onAnimationDidEnd?: Function) 

计划下一次布局要发生的动画。

@param config 表示动画相应的属性

+ duration 动画持续时间，单位是毫秒
+ create, 配置创建新视图时的动画。(参阅 Anim 类型)
+ update, 配置被更新的视图的动画。(参阅 Anim 类型)

@param onAnimationDidEnd 当动画结束的时候被调用。只在iOS设备上支持。　　
@param onError 当动画残生错误的时候被调用。只在iOS设备上支持。

> static create(duration: number, type, creationProp) 

用来创建configureNext所需的config参数的辅助函数。

#### 属性

> Types: CallExpression 
> Properties: CallExpression 
> configChecker: CallExpression 
> Presets: ObjectExpression 
> easeInEaseOut: CallExpression 
> linear: CallExpression 
> spring: CallExpression 

示例：

```js
import React, { Component } from 'react';
import {
  View,
  LayoutAnimation,
  StyleSheet,
  TouchableOpacity,
  Text
 } from 'react-native'

class Sample extends Component {
  constructor(props) {
    super(props);
    this.state = { w: 100, h: 100 };
    this._onPress = this._onPress.bind(this);
  }

  componentWillMount() {
    // 创建动画
    LayoutAnimation.spring();
  }

  _onPress() {
    // 动画更新，让视图的尺寸变化以动画形式展现
    LayoutAnimation.spring();
    this.setState({w: this.state.w + 15, h: this.state.h + 15})
  }

  render() {
    return (
      <View style={styles.container}>
        <View style={[styles.box, {width: this.state.w, height: this.state.h}]} />
        <TouchableOpacity onPress={this._onPress}>
          <View style={styles.button}>
            <Text style={styles.buttonText}>点击！</Text>
          </View>
        </TouchableOpacity>
      </View>
    );
  }
}

var styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  box: {
    backgroundColor: 'red',
  },
  button: {
    marginTop: 10,
    paddingVertical: 10,
    paddingHorizontal: 20,
    backgroundColor: 'black',
  },
  buttonText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
});

export default Sample;
```

#### requestAnimationFrame

requestAnimationFrame是一个对浏览器标准API的兼容实现，你可能已经熟悉它了。它接受一个函数作为唯一的参数，并且在下一次重绘之前调用此函数。一些基于JavaScript的动画库高度依赖于这一API。通常你不必直接调用它——那些动画库会替你管理好帧的更新。

#### react-tween-state(不推荐--用Animated来替代)

react-tween-state是一个极小的库，正如它名字（tween：补间）表示的含义：它生成一个节点的状态的中间值，从一个开始值，结束于一个到达值。这意味着它会生成这两者之间的值，然后在每次requestAnimationFrame的时候修改状态。

一个最基础的从一个值运动到另一个值的办法就是线性过渡：只需要将结束值减去开始值，然后除以动画总共需要经历的帧数，再在每一帧加到当前值上，一直到结束值位置。线性过渡有时候看起来怪异且不自然，所以react-tween-state提供了一系列常用的过渡函数，可以用于使你的动画更加自然。

这个库并未随React Native一起发布——要在你的工程中使用它，则需要先在你的工程目录下执行npm i react-tween-state --save来安装。
示例：

```js
import tweenState from 'react-tween-state';
import reactMixin from 'react-mixin'; // https://github.com/brigand/react-mixin

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { opacity: 1 };
    this._animateOpacity = this._animateOpacity.bind(this);
  }

  _animateOpacity() {
    this.tweenState('opacity', {
      easing: tweenState.easingTypes.easeOutQuint,
      duration: 1000,
      endValue: this.state.opacity === 0.2 ? 1 : 0.2,
    });
  }

  render() {
    return (
      <View style={{flex: 1, justifyContent: 'center', alignItems: 'center'}}>
        <TouchableWithoutFeedback onPress={this._animateOpacity}>
          <View ref={component => this._box = component}
                style={{width: 200, height: 200, backgroundColor: 'red',
                        opacity: this.getTweeningValue('opacity')}} />
        </TouchableWithoutFeedback>
      </View>
    )
  }
}

reactMixin.onClass(App, tweenState.Mixin);
```

#### Rebound (不推荐--用Animated来替代)

Rebound.js是一个安卓版Rebound的JavaScript移植版。它在概念上类似react-tween-state：你有一个起始值，然后定义一个结束值，然后Rebound会生成所有中间的值并用于你的动画。Rebound基于弹性物理模型，你不需要提供一个动画的持续时间，它会自动根据弹性系数、助力、当前值和结束值来计算。在React Native内部应用了Rebound，比如Navigator和WarningBox。

需要注意的是Rebound动画可以被中断——中断之后会从当前状态弹回初始值。

```js
import rebound from 'rebound';

class App extends React.Component {
  constructor(props) {
    super(props);
    this._onPressIn = this._onPressIn.bind(this);
    this._onPressOut = this._onPressOut.bind(this);
  }
  // 首先我们初始化一个spring动画，并添加监听函数，
  // 这个函数会在spring更新时调用setState
  componentWillMount() {
    // 初始化spring
    this.springSystem = new rebound.SpringSystem();
    this._scrollSpring = this.springSystem.createSpring();
    var springConfig = this._scrollSpring.getSpringConfig();
    springConfig.tension = 230;
    springConfig.friction = 10;

    this._scrollSpring.addListener({
      onSpringUpdate: () => {
        this.setState({scale: this._scrollSpring.getCurrentValue()});
      },
    });

    // 将spring的初始值设为1
    this._scrollSpring.setCurrentValue(1);
  }

  _onPressIn() {
    this._scrollSpring.setEndValue(0.5);
  }

  _onPressOut() {
    this._scrollSpring.setEndValue(1);
  }

  render() {
    var imageStyle = {
      width: 250,
      height: 200,
      transform: [{scaleX: this.state.scale}, {scaleY: this.state.scale}],
    };

    var imageUri = "img/ReboundExample.png";

    return (
      <View style={styles.container}>
        <TouchableWithoutFeedback onPressIn={this._onPressIn}
                                  onPressOut={this._onPressOut}>
          <Image source={{uri: imageUri}} style={imageStyle} />
        </TouchableWithoutFeedback>
      </View>
    );
  }
}
```
你还可以为弹跳值启用边界，这样它们不会超出，而是会缓缓接近最终值。上面的例子中可以使用this._scrollSpring.setOvershootClampingEnabled(true)来启用边界。

##### 关于setNativeProps

setNativeProps方法可以使我们直接修改基于原生视图的组件的属性，而不需要使用setState来重新渲染整个组件树。

我们可以把这个用在Rebound样例中来更新缩放比例——如果我们要更新的组件有一个非常深的内嵌结构，并且没有使用shouldComponentUpdate来优化，那么使用setNativeProps就是有用处的。

```js
// 回到上面示例的那个组件中，找到componentWillMount方法，
// 然后将scrollSpring的监听函数替换为如下代码:
this._scrollSpring.addListener({
  onSpringUpdate: () => {
    if (!this._photo) { return }
    var v = this._scrollSpring.getCurrentValue();
    var newProps = {style: {transform: [{scaleX: v}, {scaleY: v}]}};
    this._photo.setNativeProps(newProps);
  },
});

// 最后，我们修改render方法，不再通过style来传入transform（避免
// 重新渲染时产生冲突）；然后给图片加上ref引用。 
render: function() {
  return (
    <View style={styles.container}>
      <TouchableWithoutFeedback onPressIn={this._onPressIn} onPressOut={this._onPressOut}>
        <Image ref={component => this._photo = component}
               source={{uri: "https://facebook.github.io/react-native/img/ReboundExample.png"}}
               style={{width: 250, height: 200}} />
      </TouchableWithoutFeedback>
    </View>
  );
}
```
不过你没办法把setNativeProps和react-tween-state结合使用，因为更新的补间值会自动被库设置到state上——Rebound则不同，它通过onSprintUpdate函数在每一帧中给我们提供一个更新后的值。

如果你发现你的动画丢帧（低于60帧每秒），可以尝试使用setNativeProps或者shouldComponentUpdate来优化它们。你还可能需要将部分计算工作放在动画完成之后进行，这时可以使用InteractionManager。你还可以使用应用内的开发者菜单中的“FPS Monitor”工具来监控应用的帧率。

#### 导航器场景切换

Navigator使用JavaScript实现，而NavigatoIOS则是一个对于UINavigationController提供的原生功能的包装。所以这些场景切换动画仅仅对Navigator有效。为了在Navigator中重新创建UINavigationController所提供的动画并且使之可以被自定义，React Native导出了一个NavigatorSceneConfigsAPI。

```js
import { Dimensions } from 'react-native';
var SCREEN_WIDTH = Dimensions.get('window').width;
var BaseConfig = Navigator.SceneConfigs.FloatFromRight;

var CustomLeftToRightGesture = Object.assign({}, BaseConfig.gestures.pop, {
  // 用户中断返回手势时，迅速弹回  
  snapVelocity: 8,
  // 如下设置可以使我们在屏幕的任何地方拖动它
  edgeHitWidth: SCREEN_WIDTH,
});

var CustomSceneConfig = Object.assign({}, BaseConfig, {
  // 如下设置使过场动画看起来很快
  springTension: 100,
  springFriction: 1,
  // 使用上面我们自定义的手势
  gestures: {
    pop: CustomLeftToRightGesture,
  }
});
```