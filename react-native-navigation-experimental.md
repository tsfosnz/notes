## 代码分析

> http://facebook.github.io/react-native/releases/0.41/docs/navigation.html

首先说一下，这几个部分：

- NavigationTypeDefinition

这一部分是这个库自定义的类型的综合模块。

其中使用了FB flow的规则。

> https://flow.org/en/docs/

- NavigationAnimatedValue, Animated.Value
- NavigationGestureDirection, Union type, could be any of these value 'horizontal' | 'vertical'
- NavigationState, A object with [key: string] as property
- NavigationParentState, A object with [index: number, key: string, children: Array<NavigationState>] as property.

Here index will be the scene index, and key will be scene key, one parent state had many navigation state(?)

- NavigationAction, Any
- NavigationLayout, A object with:

```
height: NavigationAnimatedValue,
initHeight: number,
initWidth: number,
isMeasured: boolean,
width: NavigationAnimatedValue
```

So (height, width) is control the size of (?), and initHeight / initWidth is the latest size of the view from onLayout event, and isMeasured is a value to indicate that the onLayout happened. So the size is the recent size of the containing View.

This is used in NavigationAnimatedView, also it will set (width, height) value to be the new size, thus, the renderScene will use this value to (?)

- NavigationPosition, Animated.Value

Position will be where the scene is, its a index

- NavigationScene, A object with:

```
index: number,
isStale: boolean,
key: string,
navigationState: NavigationState
```

Each scene will have [index: where it is, isStale: not active anymore(?), key: which one(?), state: ?]

- NavigationSceneRendererProps, A object with:

```
layout: NavigationLayout, The layout of the containing view of the scenes.
navigationState: NavigationParentState, // The navigation state of the containing view.
onNavigate: NavigationActionCaller, Callback to navigation with an action.
position: NavigationPosition, The progressive index of the containing view's navigation state.
scene: NavigationScene, The scene to render.
scenes: Array<NavigationScene>, All the scenes of the containing view's.
```

A containing view (?) will have many scene, and one is the focus (?) one to render.

- NavigationPanPanHandlers, pretty straightforward.

- NavigationActionCaller, Function
- NavigationAnimationSetter, A function with

```
position: NavigationAnimatedValue,
newState: NavigationParentState,
lastState: NavigationParentState,
```

No return. ```position``` will decide which one will going to be focus, it could be for containing view.

- NavigationRenderer, A function with

```
navigationState: ?NavigationState,
onNavigate: NavigationActionCaller,
```
return React.Component. 

- NavigationReducer, A function with

```
state: ?NavigationState,
action: ?NavigationAction,
```
return NavigationState, which one will be next scene(?)

- NavigationSceneRenderer, A function with
```
props: NavigationSceneRendererProps,
```
return React.Component

- NavigationStyleInterpolator
```
props: NavigationSceneRendererProps
```
return Object. Basically from (width, height, position) to a new value. Containing view(?)

然后，开始看代码的细节。

作为一个导航组件，一般包括页面切换的水平导航，Tab 的分页导航，以及使用对话窗体的 Model 垂直导航，这也是iOS最基础的三种模式。

### 关于水平导航

为了支持如 iOS 水平导航，不考虑任何具体的内容组件，就框架而言，一定存在多个场景，每一个场景，包含一个用户内容组件，在多个场景之上，一定有一个用来保存的方式，以及控制场景变化的方式。

保存的方式，可以从上面的 NavigationSceneRendererProps 看出来，这里有唯一的保存多个场景的参数，按照注释的说明，这个参数是，场景之上的导航框架用来保存起内部的所有场景的参数。

另外，还有一个单独的 scene 参数，用来保存当前渲染的那个场景所在，可见通过这个参数，可以访问场景所在的组件，这也就是 NavigationScene 里面key的作用。每一个场景应该有一个唯一的 key 。

对于 iOS 而言，构建一个水平导航，需要首先建立 NavigationController 这样一个框架，然后，对于任何一个独立的组件 View，都可以 PUSH 到这个框架里面，从而渲染成为当前的内容组件，从框架本身的角度看，需要把这种关系保存下来，也就是堆栈的模型，当 PUSH 的时候，新的 View 在堆栈的最上方，成为当前的组件，而当 POP 的时候，View 被释放，而前面的 View 被放在堆栈的最上方，从而成为当前组件，这是一个可逆的过程。

与此同时，Model 可以覆盖在当前的 内容之上，而关闭后，前面的水平导航状态是不变的。从这个角度，Model 和水平导航组件是同时存在的。

受这种关系的启发，当前的库，也适用了一个叫做 NavigationCardStack 的组件

- NavigationCardStack

从这个组件渲染的内容来看，其渲染了一个 NavigationAnimatedView

- NavigationAnimatedView

那么来看看这个 View 到底是如何工作的。

NavigationAnimatedView 渲染了一个普通的 View，其中截取了其 Layout 变化的事件，并且可以通过属性 Style 来修改其属性，这是第一个具体的 Native View，也是展现在屏幕上的具体样子。

在这个 View 的内部使用了另一个 View，其中包括了所有的场景的内容，这是可以理解的，虽然，当前显示的场景和内容只有一个，但是，为了实现场景切换的水平动画，把场景放置在一个水平面上，其中只有当前场景位于当前的屏幕上，别的场景位于屏幕之外，是一种简单的方法。

从水平切换的角度而言，只需要三个场景在这个水平面上就可以了，也只需要三个位置参数就够了。

在这个 View 的下面，还有一个 overlay 的组件，看起来，是为了在水平的场景的内容之上覆盖一个新的 View 建立了可能，这给 Model 出现在水平场景之上建立了条件。这也是支持 Model 的直观方法。

使用这个组件的时候，会传递过来几个属性，其中一个是 renderScene，这是渲染场景的基础函数，这个函数会接受 layout 的大小的状态，以及 NavigationState，NavigationState 是一个包含 key 为属性的对象，onNavigate，虽然是必须的，但是并没有使用，position，是通过属性 NavigationParentState 传递过来的 index 的数值构建的 Animated.Value，scene，是一个 key，而另一个参数多个 scene，是 NavigationSceneReducer 组件，其参数是 [], 以及当前的 navigationParentState。

- NavigationSceneReducer

这个组件的主要是从当前的给出的场景集合里面，按照给出的 navigationParentState，来计算并给出下一步需要的场景的集合。

然后回到 CardStack 的部分。

这里，renderScene 的这个属性的具体内容，位于 CardStack 里面，其渲染了一个 NavigationCard 组件，这个组件绑定了 NavigationCardStackPanResponder 以及 Interpolate 这两个组件的参数，分类是按照当前的动画方向是水平或者垂直而定，而这个方向，是在 NavigationCardStack 里面属性中指定的。而 overlay 这个部分的渲染，也是通过属性指定的参数，并不位于 NavigationCardStack 中。

从上面的简单分析可以看到，NavigationCardStack 是水平／垂直导航的框架，里面通过 NavigationAnimatedView 来包含不同的场景，场景时经过筛选得到的，同时，负责兼听主体 View 的大小的变化，并传递这些变化给 NavigationCard，而 NavigationCard 是手势以及动画的控制组件，其渲染场景的参数，通过 NavigationCardStack来确定。

- NavigationCard

NavigationCard 渲染使用的组件是一个 Animated.View，因为其是处理手势和动画的主体。而在其中，包含了一个 SceneView 的组件，这是一个私有的组件，只是用于此。

这个组件，接受两个参数，一个就是渲染场景的函数，一个就是渲染场景的参数。

函数，是从 NavigationCardStack 传递过来的，因为 NavigationCardStack 是最外层的组件，所以，对于渲染外部的用户内容组件而言，通过其传递，是必要的。

从上面的分析可以看出，对于使用 NavigationCard 组件而言，用户需要些自己的代码，来完善其使用的方法，其中一个就是，如何把内容组件加入导航框架的方法，也就是 RNRF 使用 Scene 这样一个表示，以及 Component 这样一个参数的原因所在。

正如 FB 所说，用户需要写很多准备代码才能够使用这个库。

### Navigator

> http://facebook.github.io/react-native/releases/0.42/docs/navigator.html

Navigator 是之前的主要导航方法，在 V7 版本中，这些导航方法都不在维护，除了 NavigationIOS。

- NavigationLegacyNavigator
- NavigationLegacyNavigatorRouteStack

这两个模块是 Navigator 的主要部分。




### Navigate & Transition

导航，从一个页面切换到另一个页面，并且通过页面的动态变换，实现平滑的过度，这是导航组件最基础的表现。

这个组件的基础动态变换的实现，位于```NavigationCardStackStyleInterpolator```这个模块里面，其基本的定义了两种方式的动态变换的参数，一个就是水平方向的切换使用的动态变换参数，一个就是垂直方向上所使用的动态变换参数。

- ```forInitial``` receive a parameter, named ```NavigationSceneRendererProps```, indeed, it had two member, one is ```navigationState```, another is ```scene```, here to assume that:

1. It showed the navigationState has the current navigation stack indicator, ```index```, and scene has its own ```index``` too.

So when two index is same, the current scene is focused, it's in current screen.

So its opacity is 1, and translate is 0, otherwise, its opacity is 0 (not visible), and translate is a large number, according to comment, looks like it move this scene out of screen, somewhere.

Then it will return back this two value, opacity, and translateX: translate, translateY: translate, we know that the translateX and translateY is the scene position in current screen, when its (0, 0), it will be located in current screen, but if its something like (1000000000, 1000000000), it will be somewhere very far away.

- ```forHorizontal```, it had the same parameter type, but it can see, it will deal with:

1. layout, position, scene, at first, it will test if layout is measured, if so, it will just return the function above. Commonly layout is the size of the view, could be {x, y, width, height}

2. position, here position is the ```index``` of the navigation item (?), it could be [index - 1, index, index + 1], it is [prev, current, next], the position is changed when navigation move, so.

It will map [prev, current, next] to [1, 1, 0.3] for opacity, so when position changed from prev to current, opacity not change, when changed from current to next, it will be fade by 0.3 until not visible.

Scale, is the same, but little different, its [1, 1, 0.95], [not change, not change, 0.95 * size].

translateX, is [width, 0, -10], [just along with screen (right), just at screen (center), just out of screen a little (-10], looks like: right --> into screen --> and then --> out of screen a little.

Because its horizontal, so translateY keep 0.

When those value applied in one animated view, it appeared to be:

The position had two scene in consideration, so the change will be [index - 1 --> index], and then [index --> index + 1], when there is a scene push into stack, then this scene will be index - 1 --> index, and the prev scene will be index --> index + 1, this two scene index change this way.

| -1 | scene A (0) | empty (1) ]  --> | scene B (0) | sceneA (1) |

From the translateX, the scene will push from right to left, a new page will be in.

By using interpolate, even though it's 0 --> 1, the animation will be smoothly start from 0 to -10, could be like [0, -0.1, -0.2, .... -10], that become smooth move.

- ```forVertical``` is same thing, it will be the animation for vertical change.

To note, is that the scale is {scaleX, scaleY}, so 0.95, it means, it will go {scaleX * 0.95, scaleY * 0.95}, appear that, it will just move to a direction in certain angular a bit.

### 手势

- NavigationTypeDefinition
- NavigationAbstractPanResponder
- NavigationCardStackPanResponder

这个模块是定义了手势部分，基本上定义了移动过程中，通过position，position是一个Animated Value，从而实现了，从index的变化，而随之而来的就是，上面定义的部分的动画效果的实现。

### Card, CardStack

- NavigationCard
- NavigationAnimatedView
- NavigationCardStack

Card 是包含 Scene 的组件，Scene 可以看成最接近内容组件的组件，是内容组件的容器，表达了场景的存在。而 Card 是场景的容器，处理手势和动画的对象。因为手势和动画应用于Animated.View，所以，使用了Scene来render内容组件，而不是Scene本身，Scene是一个抽象的渲染者。

