很高兴见到你！

作为[《重学安卓》](https://xiaozhuanlan.com/kunminx)专栏配套项目的[《Jetpack MVVM 最佳实践》](https://github.com/KunMinX/Jetpack-MVVM-Best-Practice)，于 2019.10.30 上线并长期维护。

在过去的一段时间里，我们分别在各渠道的维护和交流中，收集到许多新上手的小伙伴在把 Jetpack MVVM 应用到自己项目中时，最频繁提及的问题，

考虑到这些四处分散的 Q&A 不便于新上手的小伙伴查阅，因此单独准备了本项目，点开项目就能直接从 Readme 中查看到不定期整理和更新的高频 Q&A。

（在实际项目开发中对 Jetpack MVVM 有任何疑问，可先自助查阅 持续维护并趋于成熟的《最佳实践》源码和《重学安卓》专栏文章，遇到实在不理解的困惑可在群里讨论，或在文章评论区、本项目或 [《最佳实践》项目的 issue 区](https://github.com/KunMinX/Jetpack-MVVM-Best-Practice/issues) 提交 issue）

&nbsp;

## 目录一览

（点击标题链接可页面内跳转）

- 《重学安卓》读者群 高频 Q&A TOP 5
  - <a href="#cxaztop1">TOP 1：Jetpack MVVM 下的页面通信怎么做？</a>
  - <a href="#cxaztop2">TOP 2：LiveData “数据倒灌” 是什么情况，如何解决？</a>
  - <a href="#cxaztop3">TOP 3：逻辑为什么不在 ViewModel 中写？</a>
  - <a href="#cxaztop4">TOP 4：为什么不用 LiveDataBus？</a>
  - <a href="#cxaztop5">TOP 5：Navigation replace 方式返回时，怎么恢复视图状态？</a>
- Jetpack MVVM 最佳实践 issue 高频 Q&A TOP 5
  - <a href="#zjsjtop1">TOP 1：页面 onPause 的时候，不是不该收到消息吗？</a>
  - <a href="#zjsjtop2">TOP 2：《最佳实践》项目中的 ”DataBinding” 严格模式是怎么回事？</a>
  - <a href="#zjsjtop3">TOP 3：绑定视图状态，LiveData 和 ObservableField，怎么取舍？</a>
  - <a href="#zjsjtop4">TOP 4：LiveData observe 回调走了多次，该如何处理？</a>
  - <a href="#zjsjtop5">TOP 5：将《最佳实践》的 Navigation 修改版引入到自己项目，结果还是走的 replace，怎么办？</a>
- To Be Continue ..

&nbsp;

## 《重学安卓》读者群 高频 Q&A TOP 5

&nbsp;

### <h2 id="cxaztop1">TOP 1：Jetpack MVVM 下的页面通信怎么做？</h2>

解答：通过 SharedViewModel 来完成。

追问：为什么？

解答：我们之所以选择 Application 级的 ViewModel，而不是静态变量或传统 bus 来完成 应用内页面间的消息通信（事件回调等），是考虑到：

1.**该 ViewModel 被封装在视图控制器（Activity/Fragment）的基类**，使得消息能够 **仅限于在视图控制器之间传播**，而不污染到之外的区域。
2.同时也可避免被外部的组件拿到，而造成不可预期的推送。

具体可见[《最佳实践》](https://github.com/KunMinX/Jetpack-MVVM-Best-Practice)项目中对 SharedViewModel 的使用。

&nbsp;

### <h2 id="cxaztop2">TOP 2：LiveData “数据倒灌” 是什么情况，如何解决？</h2>

解答：“数据倒灌” 现象是我全网首创的对某类现象的概括，所以网上大概搜不到这类描述。

数据倒灌是 **专指** 在 页面通信（事件回调）的场景下，通过 SharedViewModel 的 LiveData 给当前页通知过一次，并返回上一页，下次再进入当前页时重复收到推送的情况。

目前[《最佳实践》](https://github.com/KunMinX/Jetpack-MVVM-Best-Practice)项目中通过 EventLiveData 解决了这类问题，具体可查看最新源码。

|                         Event 包装器                         |                           重写底层                           |                        EventLiveData                         |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![WechatIMG164.jpeg](https://i.loli.net/2020/06/12/TJpv89LwYVegd4O.jpg) | ![1921591637865_.pic_hd.jpg](https://i.loli.net/2020/06/12/ACIjucp2SMbzOv6.jpg) | ![1911591637864_.pic_hd.jpg](https://i.loli.net/2020/06/12/WijX7JA3kqKP1Fn.jpg) |

&nbsp;

### <h2 id="cxaztop3">TOP 3：逻辑为什么不在 ViewModel 中写？</h2>

解答：Jetpack MVVM 主要遵循 **数据驱动** 和 **关注点分离** 这两大特性，

其中关注点分离 是通过 “最小知道原则” 来体现：

**UI 逻辑在视图控制器（Activity / Fragment）中写**，

**业务逻辑在数据层（例如 DataRepository）写**。

ViewModel 作为 视图控制器 和 数据层 沟通的桥梁，其自身应保持轻量，以胜任 “承上启下” 的角色（保持整体框架的 单向依赖）。

而且，就像认识其他问题一样，“逻辑该在 Activity 中写还是 ViewModel 中写”，

#### 要搞清楚这个问题，我们 仍然需要首先搞清楚，这件事的背景是什么 ——

> 是在多人协作的软件工程的背景下。

👆👆👆 划重点

这意味着什么呢？意味着，**一旦** 你将 UI 逻辑放在 ViewModel 中写了，**后续就不可控了**，

你的同事如果不熟悉这一套开发模式，在 “破窗效应” 的驱使下，就可能直接在 ViewModel 中取 context、取各种不该取的东西，最终内存泄漏什么的，全都来了。

综上，ViewModel 的职责边界就是帮助 Activity/Fragment 托管数据，不适合在 ViewModel 中写逻辑。

更多细节内容详见 [《有了 Jetpack ViewModel . . . 真的可以为所欲为！》](https://xiaozhuanlan.com/topic/6257931840) 中的介绍。

![WX20200612-143224@2x.png](https://i.loli.net/2020/06/12/G6BUshkH5m9uJyZ.png)

&nbsp;

### <h2 id="cxaztop4">TOP 4：为什么不用 LiveDataBus？</h2>

解答：原因同上。

不使用 LiveDataBus 是因为，我们是以 在 多人协作、页面繁杂的 软件工程 为背景来谈论架构设计的。在这样的背景下，任何微不足道的隐患，都可能被无限放大。

bus 自身 **缺乏唯一可信源的理念约束** 以及 **难以追溯事件源对象**，应彻底从项目中移除，以免团队新手的误用乃至滥用。 

具体缘由可参考 [《LiveData 鲜为人知的 身世背景 和 独特使命》](https://xiaozhuanlan.com/topic/0168753249) 中的介绍。

与此同时，尽可能使用 单例或全局 ViewModel 来托管 liveData，这样调试时能根据内存中的 liveData 对象找到事件源。LiveDataBus 这种通过 tag 来标记的，难以找到。

&nbsp;

### <h2 id="cxaztop5">TOP 5：Navigation replace 方式返回时，怎么恢复视图状态？</h2>

解答：Navigation 的 FragmentNavigator，官方写法是通过 replace 来启动新 Fragment，这可能造成返回时重绘页面等问题，对此有两种办法，一种是重写 FragmentNavigator，使之通过 show hide 来启动新 Fragment，另一种是在 onCreateView 中复用上一次实例化好的 View。

具体操作和注意事项可参考 [《就算不用 Jetpack Navigation，也请务必领略的声明式编程之美！》](https://xiaozhuanlan.com/topic/5860149732) 文末的详细补充，以及我和 [Flywith24](https://github.com/Flywith24) 在 [《我的碎片很听话，你的 Fragment 有自己的想法》](https://xiaozhuanlan.com/topic/0937256481) 评论区 22 楼关于 replace 方式返回时视图状态恢复的讨论。

&nbsp;

## Jetpack MVVM 最佳实践 issue 高频 Q&A TOP 5：

&nbsp;

### <h2 id="zjsjtop1">TOP 1：页面 onPause 的时候，不是不该收到消息吗？</h2>

解答：看到网上有不少 以讹传讹的网文 传播 “页面 onPause 时不会收到 LiveData 通知” 等不实观点，给读者们徒添困扰、耽误大量时间，特此辟谣：

事实恰恰相反，onPause 可以收到，而 onStart 不是所有场景都能收到（截至 2020.2，Activity 能，Fragment 不能） ——

只有 onResume 和 onPause 是介于 STARTED、RESUMED 状态之间，也即只有这两个生命周期节点 100% 确定能够收到 LiveData 的推送。

具体缘由详见专栏 [《为你还原一个真实的 Jetpack Lifecycle》](https://xiaozhuanlan.com/topic/3684721950) 文末 最新补充

![WechatIMG3901.jpeg](https://i.loli.net/2020/02/27/zZ1VgmkWTQEqbUO.jpg) 

&nbsp;

### <h2 id="zjsjtop2">TOP 2：《最佳实践》项目中的 ”DataBinding” 严格模式是怎么回事？</h2>

解答：“严格模式” 是我基于对 “数据驱动” 的本质的理解，而全网首创的 软件工程安全的 “纯粹数据驱动” 的写法。换言之，只要遵循 “严格模式”，就可以确保 **100% 解决视图调用的一致性问题**（安全性等价于基于函数式编程思想的 Jetpack Compose），避免在多布局等背景下滋生的各种 null 安全情况的发生。

关于 “数据驱动” 的本质，可详见 [《从 被误解 到 真香 的 Jetpack DataBinding！》](https://xiaozhuanlan.com/topic/9816742350) 和 [《是 事关软件工程安全 的 数据驱动 UI 框架 上车指南》](https://xiaozhuanlan.com/topic/2356748910) 中全网独家提供的深度解析。

&nbsp;

### <h2 id="zjsjtop3">TOP 3：为什么 MainActivityViewModel 中使用 LiveData 绑定视图状态，而其他 State-ViewModel 使用 ObservableField？</h2>

解答：**ObservaleField 有防抖的特点**，要记住这个特点，然后根据情况选择使用。

比如 PureMusic 中通知抽屉打开，用 `ObservaleField<Boolean>` 不合适，而 LiveData 合适，
因为 ObservaleField 防抖，第一次 set true，就有 true 为 value 了，第二次再 set true，就不 notify 视图刷新了（具体见 ObservaleBoolean 的 set 方法实现）

防抖可以避免重复刷新 以减少不必要的性能开销，所以看情况选择 ObservaleField 或 LiveData。

更多细节内容详见 [《从 被误解 到 真香 的 Jetpack DataBinding！》](https://xiaozhuanlan.com/topic/9816742350) 文末及评论区中的补充。

&nbsp;

### <h2 id="zjsjtop4">TOP 4：LiveData observe 回调走了多次，该如何处理？</h2>

解答：（注意此处所指的情况不同于 ”数据倒灌“）

考虑到此前有多位小伙伴私下询问过 LiveData “重复回调”的问题，这里额外做个明示：

LiveData 是被设计为，支持从 ViewModel、单例等唯一可信源 完成数据的一对多分发，因而其内部的观察套路 **并非 “一对一”的 观察者模式，而是 “一对多” 的 发布-订阅模式**，我在 2018 年自主设计并开源的 [VIABUS 架构](https://github.com/KunMinX/VIABUS-Architecture) 也是采取这种模式，内部通过 Map 来维护订阅者。

所以正常情况下，对于 一个 LiveData 实例，在同一个页面中只该注册一次观察、请勿在 RecyclerView Adapter 的 onBindViewHolder 等处注册，避免导致重复注册多个订阅者，从而不可预期地在每次请求后 “收到多次推送”。

更多完整的提示可参见 [《LiveData 鲜为人知的 身世背景 和 独特使命》](https://xiaozhuanlan.com/topic/0168753249) 文末的最新补充。

&nbsp;

### <h2 id="zjsjtop5">TOP 5：将《最佳实践》的 Navigation 修改版引入到自己项目，结果还是走的 replace，怎么办？</h2>

解答：请移除自己项目中引入的 navigation.fragment gradle 引用，不然可能会覆盖来自 architecture module 下的那些。
并且，请确保 navigation.fragment 被移入自己项目时，和原来 architecture module 中一样，使用完整的 com.androidX 的包名路径。

&nbsp;

## 版权声明

本文以 [CC 署名-非商业性使用-禁止演绎 4.0 国际协议](https://creativecommons.org/licenses/by-nc-nd/4.0/deed.zh) 发行。

Copyright © 2019-present KunMinX

![](https://user-gold-cdn.xitu.io/2020/5/22/1723c10d41f87699?w=88&h=31&f=png&s=1566)
