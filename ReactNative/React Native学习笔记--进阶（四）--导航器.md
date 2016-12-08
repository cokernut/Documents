---
title: React Native 学习笔记--进阶（四）--导航器
date: 2016-12-08
tags: React Native
categories: React Native
description:
---
React Native 进阶（四）--导航器
<!--more-->

## 导航器对比

如果你刚开始接触，那么直接选择Navigator就好。如果你只针对iOS平台开发，并且想和系统原生外观一致，那么可以选择NavigatorIOS。如果你想更好地管理导航栈，那么应该尝试一下NavigationExperimental。

### Navigator

Navigator使用纯JavaScript实现了一个导航栈，因此可以跨平台工作，同时也便于定制。

Navigator可以在renderScene方法中根据当前路由渲染不同的组件。默认情况下新的场景会从屏幕右侧滑进来，但你也可以通过configureScene方法来管理这一行为。你还可以通过navigationBar属性来配置一个跨场景的导航栏。但我们不推荐使用跨场景的navigationBar，它的代码逻辑维护起来很困难！建议自己在场景中用View实现自定义的导航栏。

#### 方法

如果你得到了一个navigator对象的引用，则可以调用许多方法来进行导航：

+ getCurrentRoutes() - 获取当前栈里的路由，也就是push进来，没有pop掉的那些。
+ jumpBack() - 跳回之前的路由，当然前提是保留现在的，还可以再跳回来，会给你保留原样。
+ jumpForward() - 上一个方法不是调到之前的路由了么，用这个跳回来就好了。
+ jumpTo(route) - 跳转到已有的场景并且不卸载。
+ push(route) - 跳转到新的场景，并且将场景入栈，你可以稍后跳转过去
+ pop() - 跳转回去并且卸载掉当前场景
+ replace(route) - 用一个新的路由替换掉当前场景
+ replaceAtIndex(route, index) - 替换掉指定序列的路由场景
+ replacePrevious(route) - 替换掉之前的场景
+ resetTo(route) - 跳转到新的场景，并且重置整个路由栈
+ immediatelyResetRouteStack(routeStack) - 用新的路由数组来重置路由栈
+ popToRoute(route) - pop到路由指定的场景，在整个路由栈中，处于指定场景之后的场景将会被卸载。
+ popToTop() - pop到栈中的第一个场景，卸载掉所有的其他场景。

这些都是navigator可以用的public method,就是跳转用的，里面有些带参数的XXX(route)，这个route参数是什么呢，这个route就是:  
renderScene={(route, navigator) => 
这里的route，最基本的route就是:

```js
let route = {
  component: SampleComponent
}
```

#### 属性

> configureScene function 

可选的函数，用来配置场景动画和手势。会带有两个参数调用，一个是当前的路由，一个是当前的路由栈。然后它应当返回一个场景配置对象

> (route, routeStack) => Navigator.SceneConfigs.FloatFromRight 

+ Navigator.SceneConfigs.PushFromRight (默认)
+ Navigator.SceneConfigs.FloatFromRight
+ Navigator.SceneConfigs.FloatFromLeft
+ Navigator.SceneConfigs.FloatFromBottom
+ Navigator.SceneConfigs.FloatFromBottomAndroid
+ Navigator.SceneConfigs.FadeAndroid
+ Navigator.SceneConfigs.HorizontalSwipeJump
+ Navigator.SceneConfigs.HorizontalSwipeJumpFromRight
+ Navigator.SceneConfigs.VerticalUpSwipeJump
+ Navigator.SceneConfigs.VerticalDownSwipeJump

> initialRoute object 

定义启动时加载的路由。路由是导航栏用来识别渲染场景的一个对象。initialRoute必须是initialRouteStack中的一个路由。initialRoute默认为initialRouteStack中最后一项。

> initialRouteStack [object] 

提供一个路由集合用来初始化。如果没有设置初始路由的话则必须设置该属性。如果没有提供该属性，它将被默认设置成一个只含有initialRoute的数组。

> navigationBar node 

可选参数，提供一个在场景切换的时候保持的导航栏。

> navigator object 

可选参数，提供从父导航器获得的导航器对象。

> onDidFocus function 

每当导航切换完成或初始化之后，调用此回调，参数为新场景的路由。

> onWillFocus function 

会在导航切换之前调用，参数为目标路由。

> renderScene function 

必要参数。用来渲染指定路由的场景。调用的参数是路由和导航器。

```js
(route, navigator) =><MySceneComponent title={route.title} navigator={navigator} />
```

> sceneStyle View#style 

将会应用在每个场景的容器上的样式。

#### 例子

> Navigator.js

```js
import React from 'react';
import {
  View,
  Navigator
} from 'react-native';
import FirstPageComponent from './FirstPageComponent';

export default class Sample extends React.Component {
  render() {
    let defaultName = 'FirstPageComponent';
    let defaultComponent = FirstPageComponent;
    return (
      <Navigator
        initialRoute={{ name: defaultName, component: defaultComponent }}
        configureScene={(route) => { // 跳转动画
          return Navigator.SceneConfigs.VerticalDownSwipeJump;
        }}
        renderScene={(route, navigator) => {
          let Component = route.component;
          // 这里有个 { ...route.params }，这个语法是把 routes.params 里的每个key 作为props的一个属性
          return <Component {...route.params} navigator={navigator} />
        }} 
      />
    );
  }
} 
```
```
第10行: 一个初始首页的component名字，比如我写了一个component叫HomeComponent，
        那么这个name就是这个组件的名字[HomeComponent]了。  
第11行: 这个组件的Class，用来一会儿实例化成 <Component />标签
第14行: initialRoute={{ name: defaultName, component: defaultComponent }} 这个指定了默认的页面，
        也就是启动app之后会看到界面的第一屏。 需要填写两个参数: name 跟 component。（注意这里填什么参数（参数名）纯粹是自定义的，
        因为这个参数也是你自己发自己收，自己在renderScene方法中处理。这个示例用了两个参数，但其实真正使用的参数只有component）  
第15，16，17行: configureScene={() => {return Navigator.SceneConfigs.VerticalDownSwipeJump;}} 这个是页面之间跳转时候的动画，
        具体有哪些动画可以看node_modules/react-native/Libraries/CustomComponents/Navigator/NavigatorSceneConfigs.js下的源代码。
最后的几行: renderScene={(route, navigator) => {let Component = route.component;return <Component {...route.params} navigator={navigator} />}}，
        这里是每个人最疑惑的，我们先看到回调里的两个参数:route, navigator。通过打印我们发现route里其实就是我们传递的name,component这两个货，
        navigator是一个Navigator的对象，为什么呢，因为它有push pop jump...等方法，这是我们等下用来跳转页面用的那个navigator对象。
        return <Component {...route.params} navigator={navigator} />  
        这里有一个判断，也就是如果传递进来的component存在，那我们就是返回一个这个component，结合前面 initialRoute 的参数，我们就知道，
        这是一个会被render出来给用户看到的component，然后navigator作为props传递给了这个component。
```
所以下一步，在这个FirstPageComponent里面，我们可以直接拿到这个 props.navigator:

> FirstPageComponent.js

```js
import React from 'react';
import {
  View,
  Navigator,
  TouchableOpacity,
  Text
} from 'react-native';

import SecondPageComponent from './SecondPageComponent';

export default class FirstPageComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      id: 2,
      user: null,
    };
  }
  _pressButton() {
    let _this = this;
    const { navigator } = this.props;
    //为什么这里可以取得 props.navigator?请注意Navigator.js中:
    //<Component {...route.params} navigator={navigator} />
    //这里传递了navigator作为props
    if (navigator) {
      // 入栈~ 把SecondPageComponent页面push进栈，接着跳转到SecondPageComponent
      navigator.push({
        name: 'SecondPageComponent',
        component: SecondPageComponent,
        //这个 params 其实来自于Navigator 里的一个方法的参数
        params: {  //routes.params
          id: this.state.id,
          //从SecondPageComponent获取user
          getUser: function (user) {
            _this.setState({
              user: user
            })
          }
        }
      })
    }
  }
  render() {
    if (this.state.user) {
      return (
        <View>
          <Text>用户信息: {JSON.stringify(this.state.user)}</Text>
        </View>
      );
    } else {
      return (
        <View>
          <TouchableOpacity onPress={this._pressButton.bind(this)}>
            <Text>查询ID为{this.state.id}的用户信息</Text>
          </TouchableOpacity>
        </View>
      );
    }
  }
}
```

这个里面创建了一个可以点击的区域，点击可以跳到SecondPageComponent这个页面，实现页面的跳转。
现在来创建SecondPageComponent，并且让它可以再跳回FirstPageComponent:

> SecondPageComponent.js

```js
import React from 'react';
import {
  View,
  Navigator,
  Text,
  TouchableOpacity
} from 'react-native';

import FirstPageComponent from './FirstPageComponent';

const USER_MODELS = {
  1: { name: '小李', age: 18 },
  2: { name: '小明', age: 20 }
};

export default class SecondPageComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      id: null
    };
  }
  componentDidMount() {
    //这里获取从FirstPageComponent传递过来的参数: id
    this.setState({
      id: this.props.id
    });
  }
  _pressButton() {
    const { navigator } = this.props;
    if (this.props.getUser) {
      let user = USER_MODELS[this.props.id];
      this.props.getUser(user);
    }
    //出栈~ 把当前的页面pop掉，这里就返回到了上一个页面:FirstPageComponent了
    if (navigator) {
      navigator.pop();
    }
  }
  
  render() {
    return (
      <View>
        <Text>获得的参数(id): id={this.state.id}</Text>
        <TouchableOpacity onPress={this._pressButton.bind(this)}>
          <Text>点我跳回去</Text>
        </TouchableOpacity>
      </View>
    );
  }
}
```

#### 传递参数和获取参数

##### 传递参数到新页面

传递参数到新页面，可以通过push。比如在一个 press的事件里:

> FirstPageComponent.js

```js
...
constructor(props) {
  super(props);
  this.state = {
    id: 2
  };
}
_pressButton() {
  ...
    navigator.push({
      name: 'SecondPageComponent',
      component: SecondPageComponent,
      //这个 params 其实来自于Navigator 里的一个方法的参数
      params: {
        id: this.state.id
      }
    })
  ...
}
```

params的来历:

```js
//Navigator.js
...
<Navigator
  initialRoute={{ name: defaultName, component: defaultComponent }}
  configureScene={() => {
    return Navigator.SceneConfigs.VerticalDownSwipeJump;
  }}
  renderScene={(route, navigator) => {
    let Component = route.component;
    if(route.component) {
        //这里有个 { ...route.params }
        return <Component {...route.params} navigator={navigator} />
    }
  }} 
/>
...
```

{ ...route.params }语法是把 routes.params 里的每个key 作为props的一个属性:

```js
//FirstPageComponent.js
...
navigator.push({
    name: 'SecondPageComponent',
    component: SecondPageComponent,
    params: {  //routes.params
        id: this.state.id
    }
});
...
```

这里的 params.id 就变成了 <Component id={routes.params.id} navigator={navigator}> 里的id属性(props)传递给了下一个页面。

```js
//SecondPageComponent.js
...
componentDidMount() {
  //这里获取从FirstPageComponent传递过来的参数: id
  this.setState({
    id: this.props.id
  });
}
...
render() {
  return (
    <View>
      <Text>获得的参数: id={ this.state.id }</Text>
      <TouchableOpacity onPress={this._pressButton.bind(this)}>
        <Text>点我跳回去</Text>
      </TouchableOpacity>
    </View>
  );
}
```

##### 返回参数到之前页面

返回的时候，也需要传递参数回上一个页面。但是navigator.pop()并没有提供参数，因为pop()只是从 [路由1,路由2，路由3。。。]里把最后一个路由踢出去的操作，并不支持传递参数给倒数第二个路由，这里要用到一个概念，把上一个页面的实例或者回调方法，作为参数传递到当前页面来，在当前页面操作上一个页面的state:

比如FirstPageComponent传递id到SecondPageComponent，然后SecondPageComponent返回user信息给FirstPageComponent：

```js
//FirstPageComponent.js
...
constructor(props) {
  super(props);
  this.state = {
    id: 2,
    user: null,
  }
}
...
_pressButton() {
    let _this = this;
    ...
      params: {  //routes.params
        id: this.state.id,
        //从SecondPageComponent获取user
        getUser: function (user) {
          _this.setState({
            user: user
          })
        }
      }
...
render() {
  if (this.state.user) {
    return (
      <View>
        <Text>用户信息: {JSON.stringify(this.state.user)}</Text>
      </View>
    );
  } else {
    return (
      <View>
        <TouchableOpacity onPress={this._pressButton.bind(this)}>
          <Text>查询ID为{this.state.id}的用户信息</Text>
        </TouchableOpacity>
      </View>
    );
  }
}
```

```js
//SecondPageComponent.js
...
const USER_MODELS = {
  1: { name: '小李', age: 18 },
  2: { name: '小明', age: 20 }
};

export default class SecondPageComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      id: null
    };
  }
  componentDidMount() {
    //这里获取从FirstPageComponent传递过来的参数: id
    this.setState({
      id: this.props.id
    });
  }
  _pressButton() {
    const { navigator } = this.props;
    if (this.props.getUser) {
      let user = USER_MODELS[this.props.id];
      this.props.getUser(user);
    }
    //出栈~ 把当前的页面pop掉，这里就返回到了上一个页面:FirstPageComponent了
    if (navigator) {
      navigator.pop();
    }
  }
}
...
```

## NavigatorIOS

如果你只针对iOS平台开发，那么可以考虑使用NavigatorIOS。它是基于 UINavigationController封装的。

```js
<NavigatorIOS
  initialRoute={{
    component: MyScene,
    title: 'My Initial Scene',
    passProps: { myProp: 'foo' },
  }}
/>
```
用法类似Navigator，NavigatorIOS也使用路由对象来描述场景，但有一些重要区别。其中要渲染的组件在路由对象的component字段中指定，要给目标组件传递的参数则写在passProps中。被渲染的component都会自动接受到一个名为navigator的属性，你可以直接调用此对象(this.props.navigator)的push和pop方法。  
由于NavigatorIOS使用的是原生的UIKit导航，所以它会自动渲染一个带有返回按钮和标题的导航栏。

> 你还可以看看[react-native-navigation](https://github.com/wix/react-native-navigation)，这是一个第三方的组件，旨在于提供原生的跨平台的导航组件。

## NavigationExperimental

Navigator和NavigatorIOS都是有状态的组件。如果你在app中多处使用这些组件，那么维护工作就会变得非常麻烦。NavigationExperimental以不同的方式实现了导航，它可以使用任何视图来作为导航视图，同时还用到了规约函数（reducer）自顶向下地管理状态。正如名字中的Experimental(实验)所示，这一组件的整体实现具有一定的实验性，但仍然建议你尝试一下用它去更好地管理应用的导航。

```js
<NavigationCardStack
  onNavigateBack={onPopRouteFunc}
  navigationState={myNavigationState}
  renderScene={renderSceneFun}
/>
```

引入NavigationExperimental的步骤和React Native中的其他组件一样。在引入此组件之后，还可以进一步解构其中一些有用的子组件，比如这里我们会从中解构NavigationCardStack和 NavigationStateUtils这两个子组件。

```js
import React, { Component } from 'react';
import { NavigationExperimental } from 'react-native';

const {
  CardStack: NavigationCardStack,
  StateUtils: NavigationStateUtils,
} = NavigationExperimental;
```
NavigationExperimental的实现机制与Navigator和NavigatorIOS有所不同，用它来构建导航栈还需要一些额外的步骤。

### 第一步：定义初始状态和根容器

首先创建一个新组件，我们会把它作为根容器，并在这里定义初始状态。导航栈会定义在navigationState字段中，其中也包含了初始的路由定义：

```js
import React, { Component } from 'react';
import { NavigationExperimental } from 'react-native';

const {
  CardStack: NavigationCardStack,
  StateUtils: NavigationStateUtils,
} = NavigationExperimental;

class Sample extends Component {
  constructor(props, context) {
    super(props, context);
    this.state = {
      // 定义初始的导航状态
      navigationState: {
        index: 0, // 现在是第一页（索引从0开始）
        routes: [{key: '最初的场景'}], // 初始仅设定一个路由
      },
    };
    // 稍后再补充此函数的实现细节
    this._onNavigationChange = this._onNavigationChange.bind(this);
  }
  _onNavigationChange(type) {
    // 稍后再补充此函数的实现细节
  }
  _exit() {
    //exit()实现
  }
  render() {
    return (
      <Text>这是一段占位的文字。稍后会在这里渲染导航。</Text>
    );
  }
}
```
现在我们定义了一个有状态的组件，暂时是无用的。我们的初始状态包含了一个路由对象，以及当前页面的索引值。但是这看起来跟Navigator的初始路由定义好像没什么区别！回忆一下navigator对象提供了push和pop操作，看起来也非常直观。但是前面我们说过了，现在我们会在根容器上使用规约函数来管理状态，下面继续。

### 第二步：规约导航状态

NavigationExperimental内置了一些有用的规约函数（reducer），都放在NavigationStateUtils中。我们现在要用的两个就是push和pop了。它们接受一个navigationState对象参数，然后返回新的navigationState对象。

据此我们可以这样来编写_onNavigationChange函数，在其中判断"push"和"pop"的行为，并分别规约对应的状态。

```js
_onNavigationChange(type) {
  // 从state中解构出navigationState
  let {navigationState} = this.state;
  switch (type) {
    case 'push':
      // push一个新路由，在这里就是一个带有key属性的对象。
      // key必须要确保唯一性
      const route = {key: 'Route-' + Date.now()};
      // 调用NavigationStateUtils提供的push规约函数
      navigationState = NavigationStateUtils.push(navigationState, route);
      break;
    case 'pop':
      // 使用pop函数来弹出当前路由
      navigationState = NavigationStateUtils.pop(navigationState);
      break;
  }
  // 如果没有实际变化，则NavigationStateUtils会返回同样的`navigationState`
  // 我们只会更新确实发生变化的状态
  if (this.state.navigationState !== navigationState) {
    // 请记住更新状态必须通过setState()方法！
    this.setState({navigationState});
    // 简单讲解一下上面那一句ES6语法
    // 如果key和value的字面一样，那么可以简写成一个，等同于下面的写法：
    // this.setState({navigationState: navigationState});
  }
}
```
到这里，我们已经触碰到了NavigationExperimental的精髓所在。这里我们只处理了两种行为，实际开发中行为可能更复杂，比如可能会考虑后退（back）行为，又或者是tab间的切换过渡行为等等。

我们现在还没写初始场景和实际的导航器，不过别急，我们一步一步来。

### 第三步：定义场景

为方便起见我们先定义一个Row（行）组件。其中显示了一些文字，并带有点击事件。

```js
class TappableRow extends Component {
  render() {
    return (
      <TouchableHighlight
        style={styles.row}
        underlayColor="#D0D0D0"
        onPress={this.props.onPress}>
        <Text style={styles.buttonText}>
          {this.props.text}
        </Text>
      </TouchableHighlight>
    );
  }
}
```
现在来定义实际的场景。其中用到了一个ScrollView来显示一个垂直列表，第一行显示当前路由对象的key字段值，后两行用来点击后调用导航器的push和pop方法。

```js
class MyVeryComplexScene extends Component {
  render() {
    return (
      <ScrollView style={styles.scrollView}>
        <Text style={styles.row}>
          路由: {this.props.route.key}
        </Text>
        <TappableRow
          text="加载下一个场景"
          onPress={this.props.onPushRoute}/>
        <TappableRow
          text="返回上一个场景"
          onPress={this.props.onPopRoute}/>
      </ScrollView>
    );
  }
}
```

### 第四步：创建导航栈

我们之前已经定义了状态和管理状态的规约函数，现在可以创建导航器组件了。在写导航器的同时，我们可以使用当前路由的属性来配置场景并渲染它了。

```js
class MyVerySampleNavigator extends Component {
  // 在这里绑定一些导航用的方法
  constructor(props, context) {
    super(props, context);
    this._onPushRoute = this.props.onNavigationChange.bind(null, 'push');
    this._onPopRoute = this.props.onNavigationChange.bind(null, 'pop');
    this._renderScene = this._renderScene.bind(this);
  }
  // 现在我们终于可以使用“NavigationCardStack”来渲染场景。
  render() {
    return (
      <NavigationCardStack
        onNavigateBack={this._onPopRoute}
        navigationState={this.props.navigationState}
        renderScene={this._renderScene}
        style={styles.navigator}
      />
    );
  }
  // 根据路由来渲染场景
  // `sceneProps`的具体结构定义在`NavigationTypeDefinition`的`NavigationSceneRendererProps`中
  // 这里你可以根据路由的不同来返回不同的场景组件，我们这里为了简要说明，始终只返回这一个场景组件
  _renderScene(sceneProps) {
    return (
      <MyVeryComplexScene
        route={sceneProps.scene.route}
        onPushRoute={this._onPushRoute}
        onPopRoute={this._onPopRoute}
        onExit={this.props.onExit}
      />
    );
  }
}
```

最后把我们新做的导航器放到根容器中：

```js
class Sample extends Component {
  // 这里省略了constructor和其他的方法
  render() {
    return (
      <MyVerySampleNavigator
        navigationState={this.state.navigationState}
        onNavigationChange={this._onNavigationChange}
        onExit={this._exit}
      />
    );
  }
}
```
别忘了引入组件和样式

```js
import { NavigationExperimental, PixelRatio, ScrollView, StyleSheet, Text, TouchableHighlight } from 'react-native';

const styles = StyleSheet.create({
  navigator: {
    flex: 1,
  },
  scrollView: {
    marginTop: 64
  },
  row: {
    padding: 15,
    backgroundColor: 'white',
    borderBottomWidth: 1 / PixelRatio.get(),
    borderBottomColor: '#CDCDCD',
  },
  rowText: {
    fontSize: 17,
  },
  buttonText: {
    fontSize: 17,
    fontWeight: '500',
  },
});
```