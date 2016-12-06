---
title: React Native 学习笔记--进阶（二）
date: 2016-12-06
tags: React Native
categories: React Native
description:
---
React Native 进阶（二）
<!--more-->

## 动画

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
  constructor(props: any) {
    super(props);
    this.state = {
      bounceValue: new Animated.Value(0),
    };
  }
  render(): ReactElement {
    return (
      <Animated.Image                         // 可选的基本组件类型: Image, Text, View
        source={{uri: 'http://i.imgur.com/XMKOH81.jpg'}}
        style={{
          flex: 1,
          transform: [                        // `transform`是一个有序数组（动画按顺序执行）
            {scale: this.state.bounceValue},  // 将`bounceValue`赋值给 `scale`
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

多个动画可以通过parallel（同时执行）、sequence（顺序执行）、stagger和delay来组合使用。它们中的每一个都接受一个要执行的动画数组，并且自动在适当的时候调用start/stop。举个例子：

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

输入	-400  -300  -200  -100  -50  0  50  100  101  200  
输出	450    300   150     0   0.5  1  0.5  0    0    0