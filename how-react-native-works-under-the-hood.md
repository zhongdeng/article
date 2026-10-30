# React Native 底层是怎么运作的

> 原文：[How React Native Works Under the Hood](https://x.com/betomoedano/status/2115817995612086374)，作者：Beto（[@betomoedano](https://x.com/betomoedano)）。本文为译文。

![封面](https://pbs.twimg.com/media/HVzbeMpWcAAHk-d.png)

用 React Native，你可以用 JavaScript 描述一个界面，然后用真正的 iOS 和 Android 原生 view 把它渲染出来。这两步之间到底发生了什么，我一直想好好讲清楚，所以之前录了一个短视频系列。这篇文章把那些内容整理到了一起，配上图和代码，方便你慢慢看。

🔗 想要更好的阅读体验，可以去我的 [Code with Beto 博客](https://codewithbeto.dev/blog/how-react-native-works-under-the-hood)看这篇文章。

**Metro、Hermes、JSI、UI 线程**这些名字你大概都听过，但它们之间是怎么配合的，可能心里没什么底。下面我们会跟着一个粉色圆形，从 JSX 一路走到屏幕上，然后点它一下，再看这次更新在整个系统里是怎么流转的。

如果你用过 React 的组件、props、state 和 JSX，也上手过 React Native，读这篇就不会吃力。不需要懂 C++、Swift 或 Kotlin。我们讲的是新的渲染架构 **Fabric**。

![一张二维示意图：左边是构建阶段的 Metro 和 Hermes 编译，右边是已安装 app 里的 Hermes、React、Fabric 和原生 view。](https://pbs.twimg.com/media/HVzefykXwAAfUz2.jpg)

Metro 和 Hermes 编译器在构建机器上把发布包准备好。到了用户装好的 app 里，由 Hermes 执行代码，Fabric 负责协调原生 UI。

# 从一个圆开始

描述我们这个界面，下面这点代码就够了：

```
import { View } from "react-native";

export default function App() {
  return (
    <View
      style={{
        flex: 1,
        alignItems: "center",
        justifyContent: "center",
      }}
    >
      <View
        style={{
          width: 200,
          height: 200,
          borderRadius: 100,
          backgroundColor: "#FF2D55",
        }}
      />
    </View>
  );
}
```

外层的 View 负责把子元素居中。内层的 View 宽高相等，圆角很大，背景是粉色。

在 iOS 和 Android 上，React Native 的核心 UI 组件背后都是**原生 view**，也就是平台用来展示内容、处理交互的基本构件。我们这个圆最后会变成一个带样式的原生 view。Text、TextInput 这些组件也是同样的对应关系。[React Native 的组件指南](https://reactnative.dev/docs/intro-react-native-components)里讲了这些组件和平台之间的对应。

JavaScript 依然是以 JavaScript 的形式运行的。框架里有好几个部分一起配合，才把这份描述变成操作系统能显示的东西。

# Metro 把 JSX 变成 JavaScript

你的应用一开始只是一堆文件：组件、工具函数、import 和依赖。**Metro** 从入口文件开始顺着 import 往下找，把各个模块转换成可以运行的 JavaScript，再打包成一个 bundle。如果你用了 TypeScript，类型也会在这一步被去掉。[Metro 把这几个阶段叫做 resolution、transformation 和 serialization。](https://metrobundler.dev/docs/concepts/)

下面是我们例子里 App 函数真实的转换结果。我用 Metro 0.83.7 和 React Native 0.83.6 的 Babel preset，按 release 的配置生成的。外面那层模块包装和依赖导入省略掉了：

```
function App() {
  return /*#__PURE__*/(0, _jsxRuntime.jsx)(_reactNative.View, {
    style: {
      flex: 1,
      alignItems: "center",
      justifyContent: "center"
    },
    children: /*#__PURE__*/(0, _jsxRuntime.jsx)(_reactNative.View, {
      style: {
        width: 200,
        height: 200,
        borderRadius: 100,
        backgroundColor: "#FF2D55"
      }
    })
  });
}
```

尖括号没了。每个 JSX 元素都变成了一次 jsx 调用，参数是组件类型和一个 props 对象。子元素通过 children 这个 prop 传进去。我们写的尺寸和颜色还在，只不过现在是普通的 JavaScript 值。

_reactNative 和 _jsxRuntime 是生成出来的依赖引用。transform 这一步会把每个模块包一层；serialization 这一步会分配依赖 ID，靠这些 ID 把 bundle 里的模块串起来。这段代码只是转换后的 App 模块，**不是一个能直接跑的完整 bundle**。具体输出会因 Metro 版本和配置不同而有差别。

到这一步，我们手里只有一些 JavaScript 函数调用，并没有生成任何 Swift 或 Kotlin 的界面代码。

# Hermes 编译并运行 JavaScript

**Hermes** 是 React Native 默认的 JavaScript 引擎。在开启了 Hermes 的标准 release 构建里，Hermes 编译器会在构建时把 JavaScript bundle 编译成字节码。装到手机上的 app 里，带着的就是这份字节码，以及用来执行它的引擎。[React Native 的 Hermes 指南](https://reactnative.dev/docs/hermes)讲了 release 构建的这套配置。

字节码是二进制格式，直接当文本打开是看不到你组件的样子的。用**反汇编器（disassembler）**可以把这些指令转成我们能看懂的形式。我用 Hermes 编译器 0.14.1 编译了上面那个转换后的模块，然后反汇编了生成的 HBC 96 版本文件。下面是 App 函数里的一段：

```
GetEnvironment    r0, 0
LoadFromEnvironment r1, r0, 1
GetByIdShort      r3, r1, 1, "jsx"
LoadFromEnvironment r1, r0, 0
GetByIdShort      r2, r1, 2, "View"
NewObject         r1
NewObjectWithBuffer r4, 3, 3, 0, 0
PutNewOwnByIdShort r1, r4, "style"
```

这些指令不用背。GetByIdShort 用来读取 jsx、View 这样的属性。NewObjectWithBuffer 用事先存好的 key 和 value 创建一个对象。PutNewOwnByIdShort 给对象设置 style 属性。r0、r1 这些是引擎用来存中间值的虚拟寄存器。

同一份反汇编结果里，常量数据部分能找到这个圆的各个值：

```
Object Key Buffer (circle excerpt):
[String 10]  → width
[String 11]  → height
[String 15]  → borderRadius
[String 14]  → backgroundColor

Object Value Buffer (circle excerpt):
[int 200]
[int 200]
[int 100]
[String 1]   → #FF2D55
```

箭头是我加的标注，指向 dump 里字符串表的对应项。这是我们这个单独模块的生成结果，它仍然依赖 Metro 的模块系统和 app 的其他部分。这些指令描述的是 JavaScript 操作，不是绘制原生 UI 的命令。不同版本的 Hermes 生成的字节码也可能不一样。

这里要分清构建时和运行时。编译发生在你的开发机或者构建服务上，执行发生在设备上。装好的 release 包启动时直接跑打包进去的代码，不需要你电脑上开着 Metro server。

# React、JSI 和 Shadow Tree

app 运行起来后，React 会调用 App，然后处理它返回的东西。**React 元素**描述的是组件、它的 props 和它的子元素。它们是 JavaScript 对象，不是屏幕上看到的 view 本身。[React Native 的架构术语表](https://reactnative.dev/architecture/glossary)里有这些概念以及它们所代表的 host 组件的定义。

**Fabric** 是 React Native 的渲染系统。它把 React 描述的 UI 和平台的 view 连接起来，大部分渲染逻辑放在一个共享的 C++ 核心里。各平台共用这部分实现，行为也更容易保持一致。[Fabric 的架构概览](https://reactnative.dev/architecture/fabric-renderer)讲了为什么要有这个核心。

**JSI**，全称 JavaScript Interface，是让 JavaScript 引擎和 C++ 互相调用的 API。Fabric 就是通过它和自己的 C++ 核心通信的。JSI 不是一个单独的线程，也不是布局引擎，它只是系统里这几部分之间的接口。

## 一个简单的 JSI 例子

下面是我自己写的教学示例，用来演示这个接口。它注册了一个可以从 JavaScript 调用的 C++ 函数。注意，这不是 Fabric 渲染圆形的代码：

```
#include <jsi/jsi.h>
namespace jsi = facebook::jsi;

void installCircleRadius(jsi::Runtime& runtime) {
  auto radius = jsi::Function::createFromHostFunction(
      runtime, jsi::PropNameID::forAscii(runtime, "circleRadius"), 1,
      [](jsi::Runtime& rt, const jsi::Value&, const jsi::Value* args,
         size_t count) -> jsi::Value {
        if (count != 1 || !args[0].isNumber()) {
          throw jsi::JSError(rt, "Pass one numeric diameter");
        }
        return jsi::Value(args[0].asNumber() / 2.0);
      });
  runtime.global().setProperty(runtime, "circleRadius", std::move(radius));
}
```

createFromHostFunction 把我们的 C++ 回调包装成一个 JavaScript 函数。我们给它起个名字，声明参数个数为 1，然后挂到 runtime 的全局对象上。回调在使用参数之前会先检查实际传进来的参数。native 端的初始化代码调用 installCircleRadius 之后，JavaScript 里就可以这样用：

```
globalThis.circleRadius(200); // 100，在 C++ 里算出来的
```

这个例子我用 React Native 0.83.6 的 [JSI 头文件](https://github.com/facebook/react-native/blob/v0.83.6/packages/react-native/ReactCommon/jsi/jsi/jsi.h)做过语法检查。它只是展示接口本身；真实的 app 还需要做 native 注册和构建配置。Fabric 也是用 JSI 来暴露它自己的渲染操作。通过这个接口做同步调用，并不会自动把工作挪到后台线程去。

## C++ 里的表示

在 Fabric 的实现里能直接看到这层连接。下面这段摘自 [React Native 0.83.6 的 UIManagerBinding.cpp](https://github.com/facebook/react-native/blob/v0.83.6/packages/react-native/ReactCommon/react/renderer/uimanager/UIManagerBinding.cpp)，它通过 JSI 读取 JavaScript 传来的参数，再交给 C++ 的 createNode：

```
return valueFromShadowNode(
    runtime,
    uiManager->createNode(
        tagFromValue(arguments[0]),
        stringFromValue(runtime, arguments[1]),
        surfaceIdFromValue(runtime, arguments[2]),
        RawProps(runtime, arguments[3]),
        std::move(instanceHandle)),
    true);
```

这些参数分别是节点标识、组件名、surface 标识、props，以及对应 React 实例的 handle。binding 把它们转成 C++ 渲染器需要的类型。[UIManager.h](https://github.com/facebook/react-native/blob/v0.83.6/packages/react-native/ReactCommon/react/renderer/uimanager/UIManager.h) 里对应的声明能看出它返回的是什么：

```
std::shared_ptr<ShadowNode> createNode(
    Tag tag,
    const std::string &componentName,
    SurfaceId surfaceId,
    RawProps props,
    InstanceHandle::Shared instanceHandle) const;
```

std::shared_ptr<ShadowNode> 是指向一个 C++ Shadow Node 的共享指针。app 运行时，渲染器在内存里创建这些对象并把它们连起来。它不会给每个 JSX 组件生成一个 C++ 源文件。

![JSX 和 React 元素通过 JSI 连接到一棵 C++ Shadow Tree，树里是居中圆形对应的 host 组件。](https://pbs.twimg.com/media/HVzeuDAXEAAWAQR.jpg)

React 负责描述 UI。Fabric 通过 JSI 为其中的 host 组件创建一份 C++ 表示，也就是 Shadow Tree。像 App 这样的自定义组件不需要有自己的原生 view。

这份 C++ 表示就叫 **Shadow Tree**。它保存着渲染器准备原生 UI 所需要的信息，包括属性、子节点和布局数据。自定义组件可以返回别的组件，所以你数 JSX 标签或者组件函数的个数，是数不出 app 里有多少个原生 view 的。

下面是我们这个圆对应的 C++ 对象和它们之间关系的示意图。为了好读，prop 的写法做了简化；这不是能执行的 C++，也不是内存 dump：

```
ShadowTree (surface)
└── RootShadowNode
    └── ViewShadowNode: centering wrapper
        props: flex=1, alignItems=center,
               justifyContent=center
        └── ViewShadowNode: circle
            props: width=200, height=200,
                   borderRadius=100,
                   backgroundColor=pink
```

App 没有作为单独的 Shadow Node 出现，因为它是我们写的 React 函数组件。它的输出最终落到 View 这样的 host 组件上。接下来，渲染器需要算出这些节点的布局尺寸。

你还会经常听到别人说**“native 端”**。我也会这么叫，算是个口语化的说法，指的是 JavaScript runtime 之外的代码和平台能力，可能包括共享的 C++ 代码，也可能是各平台自己的实现。它不特指某个线程，也不特指渲染的某个阶段。

对这个例子来说，交接到这里已经很有用了：渲染器已经拿到了圆的尺寸和背景色，而且是 native 实现能直接处理的形式。

# Yoga 算出圆的大小和位置

我们的代码说的是“把子元素居中”，可屏幕需要的是一个具体位置。

**Yoga** 是 React Native 用的布局引擎。它根据布局样式和约束，计算出大小和位置。它只管布局，结果怎么显示是平台的事。Yoga 支持 CSS 布局的一个子集，主要是 Flexbox。[Yoga 的文档](https://www.yogalayout.dev/docs/about-yoga)里对它的职责写得很直白。

假设父容器的可用空间是 **393 × 852 个逻辑单位**，没有 padding。React Native 里的数值尺寸是[与像素密度无关的](https://reactnative.dev/docs/height-and-width)。子元素是 **200 × 200**。居中之后得到：

```
left = (393 - 200) / 2 = 96.5
top  = (852 - 200) / 2 = 326
```

算出来的 frame 可以简单写成下面这样。这只是为了方便看布局数值，不是 Yoga 输出的 JSON：

```
circleFrame = {
  x: 96.5,
  y: 326,
  width: 200,
  height: 200,
};
```

![一个 200×200 的粉色圆居中放在 393×852 的父容器里，水平偏移 96.5，垂直偏移 326。](https://pbs.twimg.com/media/HVze2g3XkAAMOfQ.jpg)

Yoga 根据居中样式和可用空间算出一个具体的 frame。这里的数值用的是逻辑单位，父容器没有 padding。

这些数字取决于父容器实际能用的空间。安全区域、padding、header，或者窗口尺寸不同，结果都会变。这里算一遍只是为了把居中这个操作讲直观，并不代表某台手机全屏下的真实数值。

文字的情况还要多考虑一点：布局系统可以让平台去测量文字，好知道它要占多大地方。固定尺寸的圆是我特意挑的一个更简单的起点。

# Fabric 挂载原生 view

渲染流水线大致分三个阶段：

1. **Render：** React 计算 UI 描述，渲染器准备 Shadow Tree。

1. **Commit：** 渲染器计算布局，为挂载准备好下一版的树。

1. **Mount：** 对比新旧两棵树，把需要的改动应用到平台的 view 上。

第一次挂载时，这个圆需要新建一个原生 view，配置好，再插进去。之后的更新，需要做的操作可能会少得多。官方的[渲染流水线指南](https://reactnative.dev/architecture/render-pipeline)把这两种情况都走了一遍。

![Render、commit、mount 最终产出一个原生圆形 view。只负责布局的外层容器可以被拍平，但它的定位依然会作用到子元素上。](https://pbs.twimg.com/media/HVze-sFXcAAppz-.jpg)

最终的原生层级可以比描述里的更浅。符合条件的纯布局容器会参与布局计算，但不一定需要有自己的 host view。

外层那个容器正好是个优化的例子。我们的外层 View 只是用来给圆定位的。如果一个容器符合 **view flattening（视图拍平）**的条件，渲染器就能保留它对布局的影响，同时省掉一个多余的原生 view。能不能拍平，取决于它的属性和行为。[View flattening](https://reactnative.dev/architecture/view-flattening) 能减少原生层级里不必要的嵌套。

## 拍平前后对比

下面用原生 view 层级来看一下区别。这是针对一个符合条件的纯布局容器画的示意图，不是从某台设备上抓下来的真实层级：

```
Without flattening
Root surface
└── Layout-only wrapper
    └── Circle view

With flattening
Root surface
└── Circle view
```

**root 下面的原生 view**
不拍平：容器 + 圆
拍平：圆

**圆在 root 坐标系里的 frame**
不拍平：x: 96.5, y: 326, 200 × 200
拍平：x: 96.5, y: 326, 200 × 200

**看到的效果**
不拍平：居中的粉色圆
拍平：居中的粉色圆

我们的 JSX 没变。即使原生层级里不需要单独的容器 view，圆的位置依然由布局描述决定。这是渲染器层面的优化，不是去改你的源码。

如果一个容器需要自己绘制东西，或者需要保留自己的原生身份，那它就得有一个真实的 host view。testID、无障碍相关的行为，或者 collapsable={false} 这些属性都可能阻止拍平。别以为嵌套的 View 都会被拍掉。

## 从布局到屏幕上的 view

现在我们有了圆的大小、位置、颜色和圆角。**Commit 记录下这份描述的下一个版本。** Yoga 的布局计算就是在这个过程中完成的；前面单独拿出来讲，只是为了让数字更好理解。

**Mount 把这份描述应用到原生界面上。** Fabric 对比新旧两棵树，算出哪些 view 需要创建、更新或删除。对我们第一次出现的这个圆，iOS 端的实现会做三件事：

1. **拿到一个 UIKit view。** 新建一个，或者复用一个空闲的。

1. **配置它。** 设置算好的位置和大小、粉色背景和圆角。

1. **把它挂到父 view 上。** 圆就成了可见 view 层级的一部分。

这些改动发生在 **UI 线程**上，也就是主线程。执行这些操作的 native 代码本来就在 React Native 里，app 运行时把我们这个圆的数值传进去就行。它不会生成新的 Swift 文件。具体实现可以看 [React Native 的 mounting manager](https://github.com/facebook/react-native/blob/v0.83.6/packages/react-native/React/Fabric/Mounting/RCTMountingManager.mm#L42-L90)。

C++ 的 Shadow Tree 还在，作为渲染器的描述继续存在。和它并存的，是一个真正的 UIKit UIView 对象，代表屏幕上的这个圆。

## 直接用 UIKit 写同一个圆

下面这段简短的 Swift 代码能把这三步看得很清楚。**这是我手写的，不是 React Native 生成的。** 父 view 已经在屏幕上，同样有 393 × 852 point 的空间：

```
import UIKit

// Teaching equivalent, written by hand. Not generated by React Native.
// rootView is already on screen and has 393 × 852 points of space.
@MainActor
func mountCircle(in rootView: UIView) -> UIView {
    let circle = UIView(
        frame: CGRect(x: 96.5, y: 326, width: 200, height: 200)
    )
    circle.backgroundColor = UIColor(
        red: 1, green: 45.0 / 255, blue: 85.0 / 255, alpha: 1
    ) // #FF2D55
    circle.layer.cornerRadius = 100
    rootView.addSubview(circle)
    return circle
}
```

UIView(...) 创建 view，并传入我们算好的 frame。接下来两行赋值让它变成粉色、变成圆形。addSubview 把它挂到父 view 上。从第一段 JSX 开始，我们要的就是这个最终效果。Apple 的 [UIView 文档](https://developer.apple.com/documentation/uikit/uiview)讲了这种父 view / 子 view 的关系。

这是个固定尺寸的例子。可用空间变化时，React Native 会用 Yoga 重新计算几何信息。

## 用 UIKit 不等于生成 Swift

React Native 的 iOS 核心 View 是通过 **Objective-C++** 来创建和更新 UIKit 对象的。这样同一个文件里既能处理 C++ 渲染器的数据，又能调用 UIKit。在 [RCTViewComponentView.mm](https://github.com/facebook/react-native/blob/v0.83.6/packages/react-native/React/Fabric/Mounting/ComponentViews/View/RCTViewComponentView.mm) 里两者都能看到。这些 native 代码已经编译进 app 了；我们的 JavaScript 只是提供要设置到 view 上的值。

**容易搞混的一点：UIKit 并不要求用 Swift**

**UIKit 是 UI 框架，Swift 和 Objective-C 是编程语言。** 两种语言都能创建和配置 UIKit 的 UIView，上面那个 Swift 圆就是例子。React Native 不写也不生成 Swift，一样能用这个框架。

Swift 在 [2014 年就已经能和 Objective-C 以及 Apple 的 UI 框架配合使用](https://www.apple.com/newsroom/2014/06/02Apple-Releases-iOS-8-SDK-With-Over-4-000-New-APIs/)，比 [React Native 在 2015 年开源 iOS 版](https://engineering.fb.com/2015/03/26/android/react-native-bringing-modern-web-techniques-to-mobile/)还早。Swift 和 C++ 的直接互操作是后来才有的，要到 [2023 年的 Swift 5.9](https://www.swift.org/blog/swift-5.9-released/)。关键区别在于：调用 UIKit 是一回事，直接和 C++ 交互是另一回事。

现在 Apple 依然支持[在一个项目里同时使用 Swift、Objective-C 和 C++](https://developer.apple.com/swift/)。以后的实现可能会更多地用 Swift，但我们现在看的这条 Objective-C++ 路径，已经把 C++ 渲染器和真正的 UIKit view 接起来了。

## 再深入一点：真实的 commit 和 Objective-C++ 调用

下面这些代码摘自 React Native 0.83.6，展示的是上面那三步背后的实现。语法看不懂也没关系，不影响你继续跟着这个圆往下走。

**commit 记录下一版的树**

前面我们把 Yoga 单独拿出来讲，是为了让布局计算看得更清楚。在 Fabric 里，这个计算是**在 commit 过程中**完成的。之后渲染器会记录 C++ Shadow Tree 的一个新版本（revision），里面带着挂载需要的信息。

C++ 里确实有一个 ShadowTree::commit(...) 函数。它的返回类型是 CommitStatus，用来报告这次 commit 的结果。它**不会**返回一段 Swift 或 UIKit 的源码字符串。正常流程下，commit 后的版本会交给 mounting 机制处理。布局计算、版本号分配、交给 mounting 这几步，都能在 [React Native 0.83.6 的 ShadowTree.cpp](https://github.com/facebook/react-native/blob/v0.83.6/packages/react-native/ReactCommon/react/renderer/mounting/ShadowTree.cpp#L250-L390) 里看到。

mounting 机制会对比树的不同版本，生成一个 **mounting transaction**：一份操作列表，外加执行这些操作需要的数据。对我们第一次出现的圆来说，在符合条件的外层容器被拍平之后，相关操作大致是这样：

```
Create: obtain a native view for the circle's tag
Insert: configure its props and layout, then attach it to the parent
```

这几行是对操作的描述，不是抓下来的 transaction dump。真实的列表里装的是 C++ 的 mutation 对象。如果是更新，带的可能是针对已有 view 的 Update 操作。树的 diff 变成 mounting transaction 的地方在 [MountingCoordinator.cpp](https://github.com/facebook/react-native/blob/v0.83.6/packages/react-native/ReactCommon/react/renderer/mounting/MountingCoordinator.cpp)。

**iOS 的 mounting 代码调用 UIKit**

在 iOS 上，React Native 有一些 **Objective-C++ 代码**，既能读取这些 C++ 对象，又能调用 Apple 的 UIKit API。这些 native 实现已经编译进 app 里了，mounting 时带着新的 props 和布局数值去执行它们。

核心 View 对应的实现是 RCTViewComponentView，它是 **UIKit 里 UIView 的子类**。下面是它真实的类声明，类体省略了：

```
@interface RCTViewComponentView : UIView <RCTComponentViewProtocol, RCTTouchableComponentViewProtocol> {
```

关键是 : UIView 这部分：说明这就是一个真正的 UIKit view 类。[声明在 RCTViewComponentView.h 里。](https://github.com/facebook/react-native/blob/v0.83.6/packages/react-native/React/Fabric/Mounting/ComponentViews/View/RCTViewComponentView.h#L25)

view registry 可以复用空闲的 component view。需要新的时候，factory 会给注册好的类分配一个实例。下面是 [RCTComponentViewFactory.mm](https://github.com/facebook/react-native/blob/v0.83.6/packages/react-native/React/Fabric/Mounting/RCTComponentViewFactory.mm#L216) 里真实的分配代码：

```
.view = [viewClass new],
```

对象有了之后，iOS 的 mounting manager 会把数据设置上去，再挂到父 view 上。下面是从 [RCTMountingManager.mm](https://github.com/facebook/react-native/blob/v0.83.6/packages/react-native/React/Fabric/Mounting/RCTMountingManager.mm#L73-L90) 里真实的 Insert 处理逻辑中挑出来的几个调用，事件、state 和收尾相关的调用省略了：

```
[newChildComponentView updateProps:newChildShadowView.props oldProps:nullptr];
[newChildComponentView updateLayoutMetrics:newChildShadowView.layoutMetrics
                          oldLayoutMetrics:EmptyLayoutMetrics];
[parentViewDescriptor.view mountChildComponentView:newChildComponentView
                                           index:mutation.index];
```

布局那个调用最终会落到 UIKit 的几何属性上。React Native 会把算好的 frame 转换一下，分别设置 view 的 center 和 bounds。这样做还能避开有 transform 时直接设置 frame 带来的歧义。下面是 [UIView+ComponentViewProtocol.mm](https://github.com/facebook/react-native/blob/v0.83.6/packages/react-native/React/Fabric/Mounting/UIView%2BComponentViewProtocol.mm#L106-L107) 里真实的代码：

```
self.center = CGPoint{CGRectGetMidX(frame), CGRectGetMidY(frame)};
self.bounds = CGRect{CGPointZero, frame.size};
```

对于这个圆这种简单背景加统一圆角的情况，view 的实现直接用它的 Core Animation layer。对应的属性赋值是：

```
layer.backgroundColor = backgroundColor.CGColor;
layer.cornerRadius = (CGFloat)borderMetrics.borderRadii.topLeft.horizontal;
```

这几行出自 [RCTViewComponentView.mm](https://github.com/facebook/react-native/blob/v0.83.6/packages/react-native/React/Fabric/Mounting/ComponentViews/View/RCTViewComponentView.mm#L961-L1008) 里相关的绘制分支。同一个文件里还有普通挂载流程中插入子 view 用的 UIKit 调用：

```
[self.currentContainerView insertSubview:childComponentView atIndex:index];
```

到这里，我们就有了一个真实的 UIView 对象，放在一个真实的 UIKit view 层级里。UIKit 和 Core Animation 用这个层级和它的 layer 把圆显示出来。C++ Shadow Tree 依然作为渲染器的表示存在；UIKit view 和它并存，是屏幕上那个原生对象。

**UI 线程**（也就是主线程）是修改原生 view 的地方。一次常规渲染里，很多工作可以跑在 JavaScript 线程上，包括用 C++ 实现的那部分。用什么语言写的，并不决定它跑在哪个线程。Fabric 还支持其他执行方式来处理优先级更高的工作，所以画一张固定的“JavaScript 线程 → shadow 线程 → UI 线程”的图反而会误导人。[React Native 的线程模型](https://reactnative.dev/architecture/threading-model)里列出了这些不同情况。

上面那个手写的 Swift 例子我用 iOS SDK 做过类型检查。它演示的是同样简单的视觉效果，并不能替代 React Native 的组件实现。

现在圆已经显示在屏幕上了。接下来我们改一下它的颜色，看看这个已有的 view 会经历什么。

# 点一下，更新的还是同一个圆

我们让这个例子能交互起来。这一版里，圆变成了一个 Pressable，背景色来自 state：

```
import { useState } from "react";
import { Pressable, View } from "react-native";

export default function App() {
  const [color, setColor] = useState("#FF2D55");

  return (
    <View
      style={{
        flex: 1,
        alignItems: "center",
        justifyContent: "center",
      }}
    >
      <Pressable
        accessibilityRole="button"
        accessibilityLabel="Change circle color"
        onPress={() => setColor("#FF9500")}
        style={{
          width: 200,
          height: 200,
          borderRadius: 100,
          backgroundColor: color,
        }}
      />
    </View>
  );
}
```

一次触摸从平台的输入系统开始。React Native 的事件和点击处理机制把它传到我们 JavaScript 里的 onPress 回调。这个回调发起一次 state 更新，React 就用新颜色重新执行这个组件。

在这次 state 更新前后，组件的位置、类型和 key 都没变。React 靠这些身份信息把新结果和之前的结果对应起来（reconcile）。如果改了类型或 key，就可能变成替换；[React 关于保留和重置 state 的指南](https://react.dev/learn/preserving-and-resetting-state)讲了这个区别。

对于我们这个稳定不变的圆，mounting transaction 里带的可以是一个针对已有 view 的 Update。iOS 端的实现会把新颜色设置到同一个 UIKit 对象和它的 layer 上。一次 render 并不意味着“把整个界面拆掉重建”。

渲染器的树还用到了**结构共享（structural sharing）**。变化的节点以及受影响的那条路径会生成新的表示，没变的分支可以在新旧版本之间共用。然后渲染器再算出下一帧界面需要对 host view 做哪些改动。这是 [state 更新流水线](https://reactnative.dev/architecture/render-pipeline#react-state-updates)的一部分。

![新旧两版 Shadow Tree 共用一个没有变化的 Other view 分支，而圆和它受影响的祖先路径因为背景变成橙色而发生变化。](https://pbs.twimg.com/media/HVzfFuvWYAEjbqo.jpg)

图里多出来的 Other view 分支是为了演示结构共享，不属于我们那个最小示例。高亮的路径发生了变化，无关的分支则可以共用。

这些都是渲染器避免多余工作的具体手段。但它们不会让所有应用自动变快：耗时的 JavaScript、很大的图片、过多的更新，这些在你自己的 app 里还是得留意。

# Fast Refresh 让开发不被打断

开发时，你改一下样式保存，就能马上看到效果，不用重新构建原生应用。**Fast Refresh** 会更新相关的 JavaScript，然后重新渲染受影响的组件。

对函数组件做兼容的修改时，state 可以保留下来。其他修改可能会重置 state，或者触发整页重载，具体看你改了什么、模块是怎么用的。改了 native 代码还是得重新做一次 native 构建。[Fast Refresh 指南](https://reactnative.dev/docs/fast-refresh)讲清楚了这些边界。

在我们的例子里，改圆的大小是 JavaScript 层面的修改。而加一个当前安装的 development build 里没有的 native 能力，就是另一类修改了。

# 更新有一条 native 边界

这个区别在发布之后同样重要。配置好的更新系统，比如 **EAS Update**，可以给已安装的 app 下发兼容的 JavaScript 和资源更新。前提是这台设备上已经装好的 native 代码要能支持新代码。Expo 用 runtime version 来匹配兼容的更新和构建。[Expo 的 runtime version 文档](https://docs.expo.dev/eas-update/runtime-versions/)讲了具体机制。

![开发阶段的 Fast Refresh 和生产环境的 JavaScript 更新是两条独立的下发路径，都依赖 app 里已经装好的 native 能力。改 native 代码需要重新构建。](https://pbs.twimg.com/media/HVzfM71XoAIO1IK.jpg)

Fast Refresh 是开发工具。生产环境的更新要走配置好的下发系统，并且必须和已安装的 native runtime 对得上。改 native 代码需要出新的构建。

修一下圆的 JavaScript 点击处理逻辑，可能在现有 runtime 范围内就能搞定。但加一个已安装 app 里没有的 native 库，就得重新做 native 构建。生产环境的更新也仍然要遵守[各平台应用商店的规则](https://docs.expo.dev/eas-update/introduction/#frequently-asked-questions-faq)，包括关于改变 app 行为的那些规定。

搞懂这套架构，最实际的好处是你能知道一个改动该落在哪一层。你能分清是 bundle 的问题还是布局的问题，是 React 的 render 还是原生 view 的更新，是改一下 JavaScript 就行，还是得出一个新的二进制包。

关注我，获取更多干货 :)

– Beto
