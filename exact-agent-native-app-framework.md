# Charlie Cheever 发布 Exact：一个用 Rust 写的 Agent Native 应用框架

> 原帖：<https://x.com/ccheever/status/2119764253242695754>  
> 作者：Charlie Cheever（[@ccheever](https://x.com/ccheever)，Expo CEO）  
> 发布时间：2026-11-09 20:09（UTC+8）  
> 说明：本文翻译了原帖，以及作者在评论区接着写的全部内容（作者自己串起来的 38 条后续推文），不包括作者回复其他人的评论。图片引用原图地址，视频保存在 `assets/exact-agent-native-app-framework/` 目录下，同时附上原视频链接。专业术语和产品名保留英文。

---

## 主帖

想给大家预览一下我最近在做的一个东西，叫 Exact。它是一个用 Rust 写的全新 Agent Native 应用框架，速度非常快，目标是让你不用做任何妥协，就能把你想要的体验做出来，分毫不差。

<video src="assets/exact-agent-native-app-framework/video1.mp4" controls width="100%"></video>

> 视频看不了的话：[原视频文件](https://video.twimg.com/amplify_video/2119752884778942464/vid/avc1/1920x1080/ZXY_7COxZpo7WPUz.mp4) · [在 X 上观看](https://x.com/ccheever/status/2119764253242695754)

[原推](https://x.com/ccheever/status/2119764253242695754)

---

## 作者在评论区的后续内容

### 1

在 Web 上，Exact 比我找到的任何一个功能完整的 Web 框架都快。它的性能跟静态网站差不多，同时又有 React 这类框架完整的动态能力。

<video src="assets/exact-agent-native-app-framework/video2.mp4" controls width="100%"></video>

> 视频看不了的话：[原视频文件](https://video.twimg.com/amplify_video/2119753305547235329/vid/avc1/512x960/jv1ywXZSgQKK-yv1.mp4) · [在 X 上观看](https://x.com/ccheever/status/2119764254861726148)

[原推](https://x.com/ccheever/status/2119764254861726148) · 2026-11-09 20:09（UTC+8）

### 2

下面是在 Socket 的 Nolan Lawson 提出的 Pokédex 基准测试上的 Web 性能表现：[nolanlawson.com/2026/10/28/the-ai-cost-of-javascript-frameworks](https://nolanlawson.com/2026/10/28/the-ai-cost-of-javascript-frameworks/)

跟 Astro 不相上下，明显领先 Next.js 和 SvelteKit。

![图片](https://pbs.twimg.com/media/HWrguPpa0AADEDX.jpg)

[原推](https://x.com/ccheever/status/2119764256631758926) · 2026-11-09 20:09（UTC+8）

### 3

再看写代码这一侧。Exact 是个新框架，还没进任何 AI 模型的权重，但它的表现已经好得出乎意料：花差不多的成本，比 Next.js 更快写出正确的结果。用 Astro 做静态站目前还是更便宜、更快，但我觉得，等 AI 模型学会了 Exact、把它学进权重里，追上来是完全有可能的。

![图片](https://pbs.twimg.com/media/HWrnLflawAA0eqA.jpg)

[原推](https://x.com/ccheever/status/2119764258900922825) · 2026-11-09 20:09（UTC+8）

### 4

在 Exact 里写 UI，用的是一门新语言，叫 Contract。Contract 从一开始就是为 AI agent 设计的。

[原推](https://x.com/ccheever/status/2119764261073481777) · 2026-11-09 20:09（UTC+8）

### 5

Contract 看起来像 Python：缩进有含义，对格式要求比较死，写出来也很精简。同时它又像 Rust 一样显式、强类型，把复杂度提前到编写阶段解决。你的 Contract 代码既描述 UI（就像在 React 里那样），也描述 state 和数据流（就像用 Redux、Zustand 或 TanStack Query 那样）。

![图片](https://pbs.twimg.com/media/HWrhViuaUAAa-yW.jpg)

[原推](https://x.com/ccheever/status/2119764262960967835) · 2026-11-09 20:09（UTC+8）

### 6

Contract 是一门受约束的语言，所以能被完整地分析。JS 框架里很多容易导致性能问题、或者让分析变得困难的坑（footgun），在 Contract 里压根不存在：你写不了递归循环，写不了 useEffect，也写不了任意的 TypeScript/JavaScript。但真实 UI 里常见的东西，几乎都能用它高效地表达出来。

[原推](https://x.com/ccheever/status/2119764265863455157) · 2026-11-09 20:09（UTC+8）

### 7

因为 Contract 是新语言，agent 一上来只会去猜它的规则和语法。与其让 agent 先啃一本长长的手册、把每个 context window 都塞满，不如让 Contract 编译器返回有用的错误信息，在 agent 需要的时候现场教它怎么完成它正想做的事。有针对性的错误信息，是让 agent 只读它需要的那部分文档、成本最低的办法之一。

![图片](https://pbs.twimg.com/media/HWrhpl2bcAEMBpN.jpg)

[原推](https://x.com/ccheever/status/2119764267578884237) · 2026-11-09 20:09（UTC+8）

### 8

在 Exact 里，agent 能控制时钟，这让测试和排查都更快、更容易。agent 不用先 sleep 一会儿、再祈祷动画已经播完，而是直接把时钟往前拨，然后读结果。这样它截的图、读到的 state，就不会跟定时器或过渡动画抢时间。这也缩小了 agent 排查“为什么会这样 / 为什么没这样”时要检查的可能性。类似这种让 agent 更容易构建和调试的设计还有很多。

[原推](https://x.com/ccheever/status/2119764270221328675) · 2026-11-09 20:09（UTC+8）

### 9

Exact 还让 agent 能通过多种方式查看应用的状态，把反馈闭环跑通：截图和录屏、查看 UI 树的抽象状态、查询布局属性，等等。

[原推](https://x.com/ccheever/status/2119764272423346677) · 2026-11-09 20:09（UTC+8）

### 10

因为 Contract 在编译期就清楚你的数据是怎么组织、彼此怎么关联的，所以它做更新时可以非常精准、高效。没有 virtual DOM，没有重新渲染，也没有单独的 hydration。这就是为什么 Exact 能像 Astro 一样快地交付一个静态网站，之后又能像 React 一样动态更新。

[原推](https://x.com/ccheever/status/2119764274310676551) · 2026-11-09 20:09（UTC+8）

### 11

改了 Contract 代码之后，应用几毫秒内就能实时更新，这些改动也能通过 OTA 更新推送出去。靠着 Contract 的这些特性，Exact 既有基于 Rust 的原生方案的速度，又有 Web 应用那样的迭代速度和灵活性。

[原推](https://x.com/ccheever/status/2119764277003423799) · 2026-11-09 20:09（UTC+8）

### 12

比 UI 更深一层的应用逻辑，在 Exact 里你可以用 Rust 写，也可以用 TypeScript 写。如果你在 Shopify 或 Linear 这种对“纯原生”多少有点信仰的公司，你可以整个 app 都用 Rust 写，连 JS VM 都不用打包进去，从头到尾都是纯原生。不过对大多数真实的 app 来说，我发现大部分业务逻辑用 TS 写其实更实际。

[原推](https://x.com/ccheever/status/2119764279239049661) · 2026-11-09 20:09（UTC+8）

### 13

你也可以混着用。比如，凡是测出来是性能热点的部分都用 Rust 写，其他部分用 TS。就算首屏的数据来自 TypeScript，Exact 也能不等 JS VM 就先把第一帧画出来。

[原推](https://x.com/ccheever/status/2119764281336160626) · 2026-11-09 20:09（UTC+8）

### 14

我确实认为，大一点的 app 用 TypeScript 写更合理，因为你（还有你的 agent）在上面持续开发、反复迭代的能力，最后是有回报的。我有个猜想：这正是 Claude Code CLI（TS）现在比 Codex CLI（Rust）精致得多、用起来舒服得多的主要原因之一。不管怎么说，在 Exact 里你总是可以自己选。Rust 和 TypeScript 都既能跑在主线程，也能跑在别的线程上，这个你也能控制。

[原推](https://x.com/ccheever/status/2119764284519641465) · 2026-11-09 20:09（UTC+8）

### 15

Contract 的改动基本只受网络速度限制，一般 10-20ms，而且通常能保住 state。TypeScript 的改动一般不到 100ms。Rust 的更新会编译成 wasm，可以热重载。在一些原生平台上可以直接换入新的动态库；不支持的平台，你可以先跑新代码的 wasm 版本，等重新构建完再换。

<video src="assets/exact-agent-native-app-framework/video3.mp4" controls width="100%"></video>

> 视频看不了的话：[原视频文件](https://video.twimg.com/amplify_video/2119756049020575744/vid/avc1/1920x1004/vimeofsrU3vFuFOV.mp4) · [在 X 上观看](https://x.com/ccheever/status/2119764286692347941)

[原推](https://x.com/ccheever/status/2119764286692347941) · 2026-11-09 20:09（UTC+8）

### 16

当然，你也可以用 Swift、Obj-C、Kotlin、HTML，或者目标平台上的其他原生语言写代码。和几乎所有框架一样，Exact 也有 escape hatch（逃生舱）：你可以划出一块矩形区域，整个交给自己的原生代码接管。但 Exact 还多给了一层控制，叫 “Access Hatches”。框架只要在目标平台上生成了某种节点，比如一个 UIKit 的 UIView，你想要的话都能直接拿到它，然后写 Swift 代码给它加个属性，或者补一个同步的原生回调，等等。Exact 的一个核心目标，就是让你始终觉得自己手里有足够精确的控制力，能把结果做成你想要的样子，分毫不差。

[原推](https://x.com/ccheever/status/2119764289817075890) · 2026-11-09 20:09（UTC+8）

### 17

举个例子：Apple 官方提供的 Liquid Glass 示例 app 有个 bug，长按一个 tile 时，展开的 tile 会被裁掉一截，松手时还会闪一下白。而在 Exact 版本里，每个原生元素你都能控制，所以很容易实现正确的行为。这是个小细节，但我做这个框架，就是为了那些在意把这种细节做对的人（这个问题是 [@JI](https://x.com/JI) 发现的）。

<video src="assets/exact-agent-native-app-framework/video4.mp4" controls width="100%"></video>

> 视频看不了的话：[原视频文件](https://video.twimg.com/amplify_video/2119756442676924416/vid/avc1/1120x1320/wgbnRzq2vQhHvCWV.mp4) · [在 X 上观看](https://x.com/ccheever/status/2119764291738083820)

[原推](https://x.com/ccheever/status/2119764291738083820) · 2026-11-09 20:09（UTC+8）

### 18

原生性能方面，大家主要关心两件事：(1) 流畅度 / fps，(2) 启动时间。

先说流畅度。下面是 Exact 和 SwiftUI 滚动同一个内容丰富的信息流的对比。可以看到 Exact 一直稳在 120fps，一帧没掉；而 SwiftUI 掉到了 30-50fps，很多帧都严重延迟。

<video src="assets/exact-agent-native-app-framework/video5.mp4" controls width="100%"></video>

> 视频看不了的话：[原视频文件](https://video.twimg.com/amplify_video/2119756599439081474/vid/avc1/1920x1080/FnMMuYwsokn07bPw.mp4) · [在 X 上观看](https://x.com/ccheever/status/2119764293973598286)

[原推](https://x.com/ccheever/status/2119764293973598286) · 2026-11-09 20:09（UTC+8）

### 19

两者接近到肉眼很难分辨，但在 fps 上，Exact 比地道的原生 UIKit 写法还略好一点。能做到这一点，是因为 Exact 处理布局和视图复用（view recycling）的方式，比大多数 UIKit 实现都更优化。只要投入足够多的工作量 / token，用 UIKit 也能做到同样的性能。但用 Exact，开箱就有。

![图片](https://pbs.twimg.com/media/HWrjhTvaoAA0vUY.jpg)

[原推](https://x.com/ccheever/status/2119764295911420197) · 2026-11-09 20:09（UTC+8）

### 20

Shopify 当初把移动端 app 从 React Native 重写成原生，原因之一就是想改善启动时间这类性能指标。下面是重写并优化过的原生版 Shopify Shop app，和用 Exact 做的同一个东西的对比。Exact 版本调用的是同一个 GraphQL API，拉的也是同样的数据。（没错，我相信 Shopify 只要花点时间和 token，也能把他们 Swift app 的启动做得一样快。）

<video src="assets/exact-agent-native-app-framework/video6.mp4" controls width="100%"></video>

> 视频看不了的话：[原视频文件](https://video.twimg.com/amplify_video/2119757112872144896/vid/avc1/1000x1188/nnrWmWS8gimUrNXZ.mp4) · [在 X 上观看](https://x.com/ccheever/status/2119764298239275185)

[原推](https://x.com/ccheever/status/2119764298239275185) · 2026-11-09 20:09（UTC+8）

### 21

启动能这么快，一部分要归功于 Contract。因为 Contract 非常显式、可分析，构建时就能把首屏的 UI，以及所有能提前算出来的数据，直接“烘焙”进 app 里，有点像原生版的静态生成 HTML。启动时，Rust 直接画出第一帧，完全不用跑你的 JavaScript。需要走网络的数据随后马上补上。另一个原因是 Exact 功能强但很精简，做大多数你想做的事，通常都不用再额外加载一大堆东西。

[原推](https://x.com/ccheever/status/2119764300361527742) · 2026-11-09 20:09（UTC+8）

### 22

Exact 在 Rust 里用 Taffy 做布局，而 Taffy 是基于 CSS 的。所以在 Exact 里写 UI 的方式大体跟 Web 差不多，agent 本来就熟悉 CSS，写起来也更顺手。样式也是同样的思路。

[原推](https://x.com/ccheever/status/2119764302081261932) · 2026-11-09 20:09（UTC+8）

### 23

Rust 代码还能用 Exact 自己的那套断行代码来测量文本。这样就能做出类似 [@chenglou](https://x.com/chenglou) 的 Pretext 示例那样的效果。

<video src="assets/exact-agent-native-app-framework/video7.mp4" controls width="100%"></video>

> 视频看不了的话：[原视频文件](https://video.twimg.com/amplify_video/2119758824764837888/vid/avc1/1920x1080/7J75j8LyGCIE0hTL.mp4) · [在 X 上观看](https://x.com/ccheever/status/2119764303482114558)

[原推](https://x.com/ccheever/status/2119764303482114558) · 2026-11-09 20:09（UTC+8）

### 24

Exact 还内置了 GPU 支持，因为丰富的动画和自绘图形越来越成为 app 和网站的标配。你还可以把实时运行的 UI 放进一个 GPU surface 里，再过一遍 shader，类似 HTML-in-Canvas。

<video src="assets/exact-agent-native-app-framework/video8.mp4" controls width="100%"></video>

> 视频看不了的话：[原视频文件](https://video.twimg.com/amplify_video/2119763506748952576/vid/avc1/1200x1350/GuC4WY4Z0bb-XtFd.mp4) · [在 X 上观看](https://x.com/ccheever/status/2119764305164030428)

[原推](https://x.com/ccheever/status/2119764305164030428) · 2026-11-09 20:09（UTC+8）

### 25

在 iOS、macOS 和 Web 上，平台本身就提供了用户熟悉、真心喜欢也确实想要的元素，所以 Exact 必须能直接用上它们。Android 平台则碎片化得多，Windows 和 Linux 上更是基本没什么统一标准，所以在这些平台上，Exact 会接管更多的像素绘制。

[原推](https://x.com/ccheever/status/2119764605983715422) · 2026-11-09 20:10（UTC+8）

### 26

在 Linux、Windows 这类平台上，Exact 自己实现了一个绘制像素的渲染器，叫 Ink。它跟 GPUI 或 egui 类似，不过是作为渲染器跑在 Exact 的布局系统下面，而且可以嵌入系统原生元素等等。对用户来说，Ink、GPUI 和 egui 的性能差不多。Ink 启动稍快一点，但 GPU 占用和峰值内存更高。

[原推](https://x.com/ccheever/status/2119764685033812348) · 2026-11-09 20:10（UTC+8）

### 27

我原本以为桌面端的优先级会比移动端和 Web 低，毕竟桌面 app 本来就少。但它玩起来太舒服，开发起来也太有意思了，我现在越来越相信，用这类框架做的原生桌面 app 会迎来一波爆发。尤其是在 Mac 上，我发现能做出比 Electron app 轻快得多、也紧凑利落得多的东西。

<video src="assets/exact-agent-native-app-framework/video9.mp4" controls width="100%"></video>

> 视频看不了的话：[原视频文件](https://video.twimg.com/amplify_video/2119764892693733376/vid/avc1/1812x1200/hXNKHZp9PzebtkUq.mp4) · [在 X 上观看](https://x.com/ccheever/status/2119764993831006601)

[原推](https://x.com/ccheever/status/2119764993831006601) · 2026-11-09 20:12（UTC+8）

### 28

Exact 还自带一个 agent native 的游戏引擎，你可以用 Contract 写类似 app 的 UI，也能用代码快速改动状态。它在原生平台上基于 Metal 和 Vulkan，在 Web 上基于 WebGPU。

<video src="assets/exact-agent-native-app-framework/video10.mp4" controls width="100%"></video>

> 视频看不了的话：[原视频文件](https://video.twimg.com/amplify_video/2119765100177543168/vid/avc1/1920x1080/MyZWSAe_xhhetn8F.mp4) · [在 X 上观看](https://x.com/ccheever/status/2119765329253642520)

[原推](https://x.com/ccheever/status/2119765329253642520) · 2026-11-09 20:13（UTC+8）

### 29

你还可以用 Exact 做 TUI。由一块块矩形面板拼起来的这类界面，它很拿手。

<video src="assets/exact-agent-native-app-framework/video11.mp4" controls width="100%"></video>

> 视频看不了的话：[原视频文件](https://video.twimg.com/amplify_video/2119765403941605376/vid/avc1/1916x1336/NpsAiD2-Z1P0lyHT.mp4) · [在 X 上观看](https://x.com/ccheever/status/2119765575211889069)

[原推](https://x.com/ccheever/status/2119765575211889069) · 2026-11-09 20:14（UTC+8）

### 30

现在软件已经全面进入 agent 时代，所以 Exact 应用也默认会被 agent 使用，并提供了各种 API、a11y 元数据等来支持这件事。

[原推](https://x.com/ccheever/status/2119766001504174246) · 2026-11-09 20:16（UTC+8）

### 31

iOS 是目前最成熟的平台。Web 和 macOS 也都挺不错。Android、Windows 和 Linux 能用，但没那么成熟。最早试用的人里，有一位当天就把自己的 iOS app 迁移了过来，因为各项指标实在太有说服力了。

![图片](https://pbs.twimg.com/media/HWrsB0mbUAA5GzT.jpg)

[原推](https://x.com/ccheever/status/2119766138276217309) · 2026-11-09 20:16（UTC+8）

### 32

还在开发中，但我觉得已经可以让大家上手试试了。仓库在这里：[github.com/expo/exact](https://github.com/expo/exact)

[原推](https://x.com/ccheever/status/2119766564966973443) · 2026-11-09 20:18（UTC+8）

### 33

[@JI](https://x.com/JI) 建了一个模板 app 仓库，你可以拿它当作用 Exact 开发的起点：[github.com/expo/exact-templates](https://github.com/expo/exact-templates)

[原推](https://x.com/ccheever/status/2119766659779232155) · 2026-11-09 20:18（UTC+8）

### 34

<sub>回复 [@JI](https://x.com/JI)</sub>

这个项目一开始是我一个人做的，但最近 [@JI](https://x.com/JI) 帮了很多忙，[@eiioth](https://x.com/eiioth) [@kzzzf](https://x.com/kzzzf) [@DaehyeonMun](https://x.com/DaehyeonMun) [@nater02](https://x.com/nater02) [@sethwebster](https://x.com/sethwebster) 和 Doug Lowder 也贡献了代码。

[原推](https://x.com/ccheever/status/2119766996489535759) · 2026-11-09 20:20（UTC+8）

### 35

Expo 框架不会消失，Expo 公司会继续支持它。它现在的增长比以往任何时候都快。随着 AI 给 React Native 带来巨大的性能提升，再加上 Expo SDK 和 Expo Modules 不断改进，它也在快速变好。Exact 会接入 Expo 的云服务，用来构建、更新、监控和托管你的 app 等等。

![图片](https://pbs.twimg.com/media/HWrtN4AagAEkJnT.png)

[原推](https://x.com/ccheever/status/2119767513332621732) · 2026-11-09 20:22（UTC+8）

### 36

这是 Exact 的 pre-alpha 预览版。有些地方肯定还没做完，或者是坏的。以后正式的发布流程会是什么样，我还没定。非常欢迎大家提 issue、提功能需求、给反馈。项目以 MIT License 开源，你可以随意修改、fork 等等。

[原推](https://x.com/ccheever/status/2119767966338433381) · 2026-11-09 20:24（UTC+8）

### 37

<sub>回复 [@JI](https://x.com/JI) [@eiioth](https://x.com/eiioth) [@kzzzf](https://x.com/kzzzf) [@DaehyeonMun](https://x.com/DaehyeonMun) [@nater02](https://x.com/nater02) [@sethwebster](https://x.com/sethwebster)</sub>

你能做的最有帮助的事，大概就是让你的 agent 把它用 Exact 开发的经历写成日志（diary），提交到反馈接口（feedback endpoint）。agent 使用过程的详细记录，对找出该修什么、该改进什么来说就是金子。感谢大家来试用。

[原推](https://x.com/ccheever/status/2119768011955732656) · 2026-11-09 20:24（UTC+8）

### 38

<sub>回复 [@JI](https://x.com/JI) [@eiioth](https://x.com/eiioth) [@kzzzf](https://x.com/kzzzf) [@DaehyeonMun](https://x.com/DaehyeonMun) [@nater02](https://x.com/nater02) [@sethwebster](https://x.com/sethwebster)</sub>

上面没看到的话，仓库链接在这里：[github.com/expo/exact](https://github.com/expo/exact)

[原推](https://x.com/ccheever/status/2119768151202451731) · 2026-11-09 20:24（UTC+8）
