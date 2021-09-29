> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6955491901265051661)

> 上篇文章我给大家分享了我对架构的理解，从思想层面去讲述架构的演进过程。很多小伙伴表示还想听我讲一下对Jetpack 架构的看法，本着帮人帮到底的精神，今天我将再次动笔 尽量从本质上讲清楚Jetpack

*   **1. 有了 Lifecycle，再也不用担心生命周期同步问题**
    *   **1.1 为什么要做生命周期绑定？**
    *   **1.2 Lifecycle 解决了哪些问题？**
*   **2. LiveData 并不是只运用观察者模式**
    *   **2.1 观察者模式的优点有哪些？**
    *   **2.2 LiveData 基于观察者模式又做了哪些扩展？**
    *   **2.3 LiveData + Lifecycle 实现 1 + 1 > 2**
*   **3. ViewModel 与 LiveData 真乃天作之合**
    *   **3.1 如何优雅的实现 Fragment 之间通讯？**
    *   **3.2 由 ViewModel 担任 VM/Presenter 的好处有哪些？**
*   **4. 解除你对 DataBinding 的误解**
    *   **4.1 使用 DataBinding 的好处有哪些？**
    *   **4.2 为什么很多人说 DataBinding 很难调试？**
*   **5. Jetpack 和 MVVM 有什么关系？**
    *   **5.1 什么是 MVVM**
    *   **5.2 Jetpack 只是让 MVVM 更简单、更安全**

1. 有了 Lifecycle，再也不用担心生命周期同步问题
------------------------------

### 1.1 为什么要做生命周期绑定？

关于`Activity/Fragment`其最重要的概念就是生命周期管理，我们开发者需要在不同生命周期回调中做不同事情。比如`onCreate`做一些初始化操作，`onResume`做一些恢复操作等等等等，以上这些操作都比较单一直接去写也没有多大问题。

但有一些组件需要强依赖于`Activity/Fragment`生命周期，常规写法一旦疏忽便会引发安全问题，比如下面这个案例：

现有一个视频播放界面，我们需要做到当跳到另一个界面就暂停播放，返回后再继续播放，退出后重置播放, 常规思路：

```
#class PlayerActivity
    onCreate(){
        player.init()
    }
    onResume(){
        player.resume()
    }
    onPause(){
        player.pause()
    }
    onDestroy(){
        player.release()
    }
复制代码
```

读过我上篇文章的小伙伴可能一眼就能看出来这违背了`控制反转`，人不是机器很容易写错或者忘写，特别是`player.release()`如果忘写便会引发内存泄漏 此时我们可以基于`控制反转思想(将player生命周期控制权交给不会出错的框架)`进行改造： 第一步：

```
interface ObserverLifecycle{
    onCreate()
    ...
    onDestroy()
}
复制代码
```

首先定义一个观察者接口, 包含`Activity/Fragment`主要生命周期方法

第二步：

```
class BaseActivity{
    val observers = mutableList<ObserverLifecycle>()
    onCreate(){
        observers.forEach{
            observer.onCreate()
        }
    }
    ...
    onDestroy(){
        observers.forEach{
            observer.onDestroy()
        }
    }
}
复制代码
```

在`BaseActivity`中观察生命周期并逐一通知到`observers`的观察者

第三步：

```
class VideoPlayer : ObserverLifecycle{
    onCreate(){
        init()
    }
    ...
    onDestroy(){
        release()
    }
}
class PlayerActivity : BaseActivity{
    observers.add(videoPlayer)
}
复制代码
```

播放器实现`ObserverLifecycle`接口，并在每个时机调用相应方法。`PlayerActivity`只需将`videoPlayer`注册到`observers`即可实现生命周期同步。

其实不光`videoPlayer`，任何需要依赖`Activity`生命周期的组件 只需实现`ObserverLifecycle`接口最后注册到`Activity`的`observers`即可实现生命周期自动化管理，进而可以规避误操作带来的风险

### 1.2 Lifecycle 解决了哪些问题？

既然生命周期的同步如此重要，Google 肯定不会视而不见，虽然自定义`ObserverLifecycle`可以解决这种问题，但并不是每个人都能想到。所以 Google 就制定了一个标准化的生命周期管理工具`Lifecycle`，让开发者碰到生命周期问题自然而然的想到`Lifecycle`，就如同想在`Android`手机上新建一个界面就会想到`Activity`一样。

同时`Activity`和`Fragment`内部均内置了`Lifecycle`，使用非常简单，以 1.1 案例通过`Lifecycle`改造后如下：

```
class VideoPlayer : LifecycleObserver {
    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    fun onCreate(){
        init()
    }
    ..
    @OnLifecycleEvent(Lifecycle.Event.ON_DESTROY)
    fun onDestroy(){
        release()
    }
}
class PlayerActivity : BaseActivity{
    lifecycle.addObserver(videoPlayer)
}
复制代码
```

两步操作即可，不用我们自己向观察者`(videoPlayer)`做生命周期分发处理。

2. LiveData 并不是只运用观察者模式
-----------------------

### 2.1 观察者模式的优点有哪些？

观察者是一种常见并且非常实用的一种行为型模式，具有扩展性强、耦合性低的特性。

本文`1.1` 中 生命周期同步设计就是一个标准的观察者模式，`ObserverLifecycle`可作为观察者，`PlayerActivity`作为被观察者, 当被观察者`(PlayerActivity)`生命周期发生改变时会主动通知到观察者`(VideoPlayer)`

同时观察者在不改变代码结构的情况随意扩展, 比如`PlayerActivity`属于一个`MVP`架构，此时可以将`Presenter`实现`ObserverLifecycle`作为观察者 随后 注册到被观察者`(PlayerActivity)`中， 这样`Presenter`也可以监测到`Activity`生命周期，并且代码结构没有任何改变，符合`开闭原则(对扩展开发 修改关闭)`

### 2.2 LiveData 基于观察者模式又做了哪些扩展？

`LiveData`符合标准的观察者模式，所以它具备扩展性强、耦合性低的特性，同样它还是一个存储数据的容器，当容器数据改变时会触发观察者，即数据驱动。

数据驱动是前端开发领域非常重要的一个概念，说数据驱动之前我们先思考一个问题，为什么要改变数据？ 答案显而易见，无非是想让数据使用者感知到而已，而`LiveData`可以优雅的实现这一流程，将 `改变、通知` 两步操作合并为一步 即省事也提高了安全性.

根据 LiveData 的特性决定它非常适合去做数据驱动 UI，下面举个例子简单描述下：

```
# 需求：改变textView内容以及对应的数据，用LiveData实现方式如下
val liveData = MutableLiveData<String>()
liveData?.observe(this, Observer { value->
            textView.text = value
        })
//这一步会改变liveData值并且会触发textView重新渲染
liveData.value = "android"
复制代码
```

看起来平平无奇甚至理所当然，但它确实解决了我们前端开发的痛点，在此之前数据和 UI 都需要我们开发者单独修改，当面对十几个 View 时很难做到不漏不忘。 引入`liveData`后改变数据会自动触发 UI 渲染，将两步操作合并为一步，大大降低出错的概率 关于`数据驱动UI`上篇文章我已经做了详细描述，感兴趣的可以翻回去查看。

### 2.3 LiveData + Lifecycle 实现 1 + 1 > 2

`LiveData`在`Lifecycle`的加持下可以实现只在可见状态接收通知，说的通俗一点`Activity`执行了`onStop()`后内部的`LiveData`就无法收到通知，这样设计有什么好处？ 举个例子： `ActivityA`和`ActivityB`共享同一个`LiveData`，伪代码如下

```
class ActivityA{
    liveData?.observe(this, Observer { value->
            textView.text = value
        })
}
class ActivityB{
    liveData?.observe(this, Observer { value->
            textView.text = value
        })
}
复制代码
```

当`ActivityA`启动`ActivityB`后多次改变`liveData`值，等回到`ActivityA`时 你肯定不希望`Observer`收到多次通知而引发`textView多次`重绘。

引入`Lifecycle`后这个问题便可迎刃而解，`liveData`绑定`Lifecycle(例子中的this)`后，当回到`ActivityA`时只会取`liveData`最新的值然后做通知，从而避免多余的操作引发的性能问题

3. ViewModel 与 LiveData 真乃天作之合
------------------------------

### 3.1 Jetpack ViewModel 并不等价于 MVVM ViewModel

经常有小伙伴将`Jetpack ViewModel` 和 `MVVM ViewModel`相提并论，其实这二者根本没有在同一个层次，`MVVM ViewModel`是`MVVM`架构中的一个角色，看不见摸不着只是一种思想。 而`Jetpack ViewModel`是一个实实在在的框架用于做状态托管，有对应的作用域可跟随 Activity/Fragment 生命周期，但这种特性恰好可以充当`MVVM ViewModel`的角色，分隔数据层和视图层并做数据托管。

所以结论是`Jetpack ViewModel`可以充当`MVVM ViewModel` 但二者并不等价

### 3.2 如何优雅的实现 Fragment 之间通讯？

`ViewModel`官方定义是一个带作用域的状态托管框架，可通过指定作用域和`Activity/Fragment`共存亡，为了将其状态托管发挥到极致，Google 甚至单独为`ViewModel`开了个后门，`Activity`横竖屏切换时不会销毁对应的`ViewModel`，为的就是横竖屏能共用同一个`ViewModel`，从而保证数据的一致性。

既然是状态托管框架那`ViewModel`的第一要务 就要时时刻刻保证最新状态分发到视图层，这让我不禁想到了 LiveData，数据的承载以及分发交给`Livedata`，而`ViewModel`专注于托管`LiveData`保证不丢失，二者搭配简直是天作之合。

有了`ViewModel`加`LiveData` ，`Fragment`之间可以更优雅的通讯。比如我的开源项目中的音乐播放器（属于`单Activity多Fragment`架构），播放页和首页悬浮都包含音乐基本信息，如下图所示：

![](simpread-引入 Jetpack 架构后，你的 App 会发生哪些变化？ (1).assets/4cfd9e18392d412483b13a31b0d522cetplv-k3u1fbpfcp-watermark.awebp) 想要使两个 Fragment 中播放信息实时同步，最优雅的方式是将播放状态托管在`Activity`作用域下`ViewModel`的`LiveData`中，然后各自做状态监听，这样只有要有一方改变就能立即通知到另一方，简单又安全，具体细节可至我的开源项目中查看。

### 3.3 由 ViewModel 担任 VM/Presenter 的好处有哪些？

传统`MVVM`和`MVP`遇到最多的的问题无非就是多线程下的内存泄露，`ViewModel`可以完全规避这个问题，内部的`viewModelScope`是一个协程的扩展函数，`viewModelScope`生命周期跟随`ViewModel`对应的`Lifecycle（Activity/Fragment`），当页面销毁时会一并结束`viewModelScope`协程作用域，所以将耗时操作直接放在`viewModelScope`即刻

另外在界面销毁时会调用`ViewModel`的`onClear`方法，可以在该方法做一些释放资源的操作，进一步降低内存泄露的风险

4. 解除你对 DataBinding 的误解
-----------------------

### 4.1 使用 DataBinding 的作用有哪些？

`DataBinding`最大的优点跟唯一的作用就是`数据 UI双向绑定`，`UI和数据`修改任何一方另外一方都会自动同步, 这样的好处其实跟`LiveData`的类似，都是做数据跟 UI 同步操作，用来保证数据和 UI 一致性。其实写到这可以发现，不管是`LiveData`、`DataBinding`还是`DiffUtil`都是用来解决数据和 UI 一致性问题，可见 Google 对这方面有多么重视，所以我们一定要紧跟官方步伐

`小知识点：`

> DataBinding 包中的`ObservableField`作用跟`LiveData`基本一致，但`ObservableField`有一个去重的效果，

### 4.2 为什么很多人说 DataBinding 很难调试？

经常听一些小伙伴提`DataBinding`不好用，原因是要在 xml 中写业务逻辑不好调试，对于这个观点我是持否定态度的。并不是我同意 xml 中写业务逻辑这一观点，我觉得碰到问题就得去解决问题，如果解决问题的路上有障碍就尽量扫清障碍，而不是一味的逃避。

如`{vm.isShow ? View.VISIBLE : View.GONE}`之类的业务逻辑不写在 xml 放在哪好呢？关于这个问题我在上篇文章`Data Mapper`章节中描述的很清楚，拿到后端数据转换成本地模型 (`此过程会编写所有数据相关逻辑`)，本地模型与设计图一一对应，不但可以将视图与后段隔离，而且可以解决 xml 中编写业务逻辑的问题。

5. Jetpack 和 MVVM 有什么关系？
------------------------

### 5.1 什么是 MVVM

`MVVM`其实是前端领域一个专注于界面开发的架构模式，总共分为`View`、`ViewModel`、`Repository`三个模块 (需严格按照单一设计原则划分)

> *   `View(视图层):` 专门做视图渲染以及 UI 逻辑的处理
> *   `Repository(远程):` 代表远程仓库，从 Repository 取需要的数据
> *   `ViewModel:` Repository 取出的数据需暂存到 ViewModel，同时将数据映射到视图层

分层固然重要，但`MVVM`最核心点是通过`ViewModel`做数据驱动 UI 以及双向绑定的操作用来解决`数据/UI`的一致性问题。`MVVM`就这么些东西，千万不要把它理解的特别复杂

`双向绑定和单向驱动应该如何选择？`

当面临`TextView`之类的`View`时`单向驱动`已经完全够用了，毕竟在我们的认知里是不需要通过`TextView`显示的`文案`改变对应数据的，此时单向驱动就能保证`数据、UI`一致。 而双向绑定通常用在可交互式的`View`中，比如`EditText`内容会通过用户输入而改变的，此时需要通过双向绑定才能保证`数据、UI`一致。不管是`双向绑定`还是`单向驱动`，只要能保证`数据、UI`一致，那它就符合`MVVM`思想

其实我上篇文章也简单说过，好的架构不应该局限到某一种模式`(MVC/MVP/MVVM)`上，需要根据自己项目的实际情况不断添砖加瓦。如果你们的后端比较善变我建议引入`Data Mapper`的概念～如果你经常和同事开发同一个界面，可以试图将每一条业务逻辑封装到`use case`中，这样大概率可以解决`Git冲突`的问题.. 等等等等，总之只要能实实在在 提高 开发效率以及项目稳定性的架构就是好架构.

### 5.2 Jetpack 只是让 MVVM 更简单、更安全

`Jetpack`是 Android 官方为确立标准化开发而提供的一套框架，`Lifecycle`可以让开发者不用过多考虑 生命周期引发的一系列问题 ～ 有了 DataBinding 的支持让数据 UI 双向绑定成为了可能 ～ `LiveData`的存在解除`ViewModel`跟`Activity`双向依赖的问题....

归根到底`Jetpack`就是一套开发框架，`MVVM`在这套框架的加持之下变得更加简单、安全。

`Tips:作者公司项目引入Jetpack后，项目稳定性有着肉眼可见的提升。`

综上所述
----

*   **Lifecycle 解决了生命周期 同步问题**
*   **LiveData 实现了真正的状态驱动**
*   **ViewModel 可以让 Fragment 通讯变得更优雅**
*   **DataBinding 让双向绑定成为了可能**
*   **Jetpack 只是让 MVVM 更简单、更安全**