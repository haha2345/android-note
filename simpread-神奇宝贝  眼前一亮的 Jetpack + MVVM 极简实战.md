> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/6850037271253483534)*   自定义 RemoteMediator 实现 network + db 的混合使用 (RemoteMediator 是 Paging3 当中重要成员)
*   使用 Data Mapper 分离数据源 和 UI
*   Kotlin Flow 结合 Retrofit2 + Room 的混合使用
*   Kotlin Flow 与 LiveData 的使用
*   使用 Coil 加载图片
*   使用 ViewModel、LiveData、DataBinding 协同工作
*   使用 Motionlayout 做动画
*   App Startup 与 Hilt 的使用

PokemonGo 涉及的技术：

*   [Gradle Versions Plugin](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fben-manes%2Fgradle-versions-plugin "https://github.com/ben-manes/gradle-versions-plugin")：检查依赖库是否存在最新版本
*   [Kotlin](https://link.juejin.cn?target=https%3A%2F%2Fkotlinlang.org%2F "https://kotlinlang.org/") + [Coroutines](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FKotlin%2Fkotlinx.coroutines "https://github.com/Kotlin/kotlinx.coroutines") + [Flow](https://link.juejin.cn?target=https%3A%2F%2Fkotlin.github.io%2Fkotlinx.coroutines%2Fkotlinx-coroutines-core%2Fkotlinx.coroutines.flow%2F-flow%2F "https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/")：flow 是对 Kotlin 协程的扩展，让我们可以像运行同步代码一样运行异步代码
*   JetPack
    *   Paging3（network + db）：用到了 Paging3 中的 `RemoteMediator` 用来实现 network + db
    *   Dagger-Hilt (2.28-alpha)：依赖注入框架
    *   App Startup：设置组件初始化顺序
    *   DataBinding：以声明方式将可观察数据绑定到界面上
    *   Room：在 SQLite 上提供了一个抽象层，流畅地访问 SQLite 数据库
    *   LiveData：在底层数据库更改时通知视图
    *   ViewModel：以注重生命周期的方式管理界面相关的数据
    *   Andriod KTX：编写更简洁、惯用的 Kotlin 代码
*   项目架构
    *   MVVM 架构
    *   Repository 设计模式
    *   Data Mapper 数据映射
*   [Retrofit2 & OkHttp3](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsquare%2Fretrofit "https://github.com/square/retrofit")：用于请求网路数据
*   [Coil](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fcoil-kt%2Fcoil%2F "https://github.com/coil-kt/coil/")：基于 Kotlin 开发的首个图片加载库
*   [material-components-android](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fmaterial-components%2Fmaterial-components-android "https://github.com/material-components/material-components-android")：模块化和可定制的材料设计 UI 组件
*   [Motionlayout](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.android.com%2Ftraining%2Fconstraint-layout%2Fmotionlayout "https://developer.android.com/training/constraint-layout/motionlayout") ：MotionLayout 是一种布局类型，可帮助您管理应用中的动画
*   [Timber](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2FJakeWharton%2Ftimber "https://github.com/JakeWharton/timber"): 日志打印
*   [JProgressView](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fhi-dhl%2FJProgressView "https://github.com/hi-dhl/JProgressView") ：一个小巧灵活可定制的进度条，支持图形：圆形、圆角矩形、矩形等等

以上技术栈对应之前写的技术文章：

*   [Jetpack 最新成员 AndroidX App Startup 实践以及原理分析](https://juejin.cn/post/6844904190440013837 "https://juejin.cn/post/6844904190440013837")
*   [Jetpack 成员 Paging3 实践以及源码分析（一）](https://juejin.cn/post/6844904193468137486 "https://juejin.cn/post/6844904193468137486")
*   [Jetpack 新成员 Paging3 网络实践及原理分析（二）](https://juejin.cn/post/6844904196207345672 "https://juejin.cn/post/6844904196207345672")
*   [Jetpack 新成员 Hilt 实践（一）启程过坑记](https://juejin.cn/post/6844904198803292173?utm_source=gold_browser_extension "https://juejin.cn/post/6844904198803292173?utm_source=gold_browser_extension")
*   [Jetpack 新成员 Hilt 实践之 App Startup（二）进阶篇](https://juejin.cn/post/6844904200590065672 "https://juejin.cn/post/6844904200590065672")
*   [Jetpack 新成员 Hilt 与 Dagger 大不同（三）落地篇](https://juejin.cn/post/6845166890562617352 "https://juejin.cn/post/6845166890562617352")
*   [全方面分析 Hilt 和 Koin 性能](https://juejin.cn/post/6846687596370722823 "https://juejin.cn/post/6846687596370722823")
*   [[译][2.4K Star] 放弃 Dagger 拥抱 Koin](https://juejin.cn/post/6844904158324064269 "https://juejin.cn/post/6844904158324064269")
*   [项目中封装 Kotlin + Android Databinding](https://juejin.cn/post/6844904131803480071 "https://juejin.cn/post/6844904131803480071")
*   [为数不多的人知道的 Kotlin 技巧以及 原理解析 (一)](https://juejin.cn/post/6844904184974835720 "https://juejin.cn/post/6844904184974835720")
*   [为数不多的人知道的 Kotlin 技巧以及 原理解析 (二)](https://juejin.cn/post/6847902224467623950 "https://juejin.cn/post/6847902224467623950")

**如果之前对这些技术没有接触过，或者只是听说，对阅读本文没有什么影响**，本文会对这些技术结合着项目 PokemonGo 来分析，为了文章的简洁性，本文不会细究技术细节，因为每个技术都需要花好几篇文章才能分析清楚，我会在后续的文章去详细分析。

如何检查依赖库最新版本
-----------

在之前的文章 [再见吧 buildSrc, 拥抱 Composing builds 提升 Android 编译速度](https://juejin.cn/post/6844904176250519565 "https://juejin.cn/post/6844904176250519565") 分析过，到目前为止大概管理 Gradle 依赖提供了 4 种不同方法：

*   手动管理 ：在每个 module 中定义插件依赖库，每次升级依赖库时都需要手动更改（不建议使用）
*   使用 ext 的方式管理插件依赖库 ：这是 Google 推荐管理依赖的方法 [Android 官方文档](https://link.juejin.cn?target=https%3A%2F%2Fdeveloper.android.com%2Fstudio%2Fbuild%2Fgradle-tips%23configure-project-wide-properties "https://developer.android.com/studio/build/gradle-tips#configure-project-wide-properties")
*   Kotlin + buildSrc：自动补全和单击跳转，依赖更新时 **将重新** 构建整个项目
*   Composing builds：自动补全和单击跳转，依赖更新时 **不会重新** 构建整个项目

新版的 AndroidStudio 只支持 **ext 的方式** 和 **手动方式管理** 检查依赖库是否存在最新版本，不支持 buildSrc、gradle-wrapper 版本的检查。

![](http://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fda9a57e33444d98a4d63aaa4dca29de~tplv-k3u1fbpfcp-watermark.awebp)

满足不了 PokemonGo 项目的需求，在 PokemonGo 项目中采用 buildSrc 方式去管理所有依赖库，因为 PokemonGo 项目采用单模块结构，而且支持 **自动补全** 和 **单击跳转** 很方便，所这里用到了 [Gradle Versions Plugin](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fben-manes%2Fgradle-versions-plugin "https://github.com/ben-manes/gradle-versions-plugin") 插件去检查依赖库的最新版本，检查结果如下所示：

```
The following dependencies have later release versions:
 - androidx.swiperefreshlayout:swiperefreshlayout [1.0.0 -> 1.1.0]
     https://developer.android.com/jetpack/androidx
 - com.squareup.okhttp3:logging-interceptor [3.9.0 -> 4.7.2]
     https://square.github.io/okhttp/
 - junit:junit [4.12 -> 4.13]
     http://junit.org
 - org.koin:koin-android [2.1.5 -> 2.1.6]
 - org.koin:koin-androidx-viewmodel [2.1.5 -> 2.1.6]
 - org.koin:koin-core [2.1.5 -> 2.1.6]

Gradle release-candidate updates:
 - Gradle: [6.1.1 -> 6.5.1]
复制代码
```

会列出所有需要更新的依赖库的最新版本，并且 Gradle Versions Plugin 比 AndroidStudio 所支持的更加全面：

*   支持手动方式管理依赖库最新版本检查
*   支持 ext 的方式管理依赖库最新版本检查
*   支持 buildSrc 方式管理依赖库最新版本检查
*   支持 gradle-wrapper 最新版本检查
*   支持多模块的依赖库最新版本检查

**那么如何使用呢？只需要三步**

*   1. 将 [PokemonGo](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fhi-dhl%2FPokemonGo "https://github.com/hi-dhl/PokemonGo") 项目根目录 checkVersions.gradle 文件拷贝到你的项目根目录下面
    
*   2. 在项目的根目录 build.gradle 文件夹内添加以下代码
    
    ```
    apply from: './checkVersions.gradle'
    buildscript {
        repositories {
            google()
            jcenter()
        }
        dependencies {
            classpath "com.github.ben-manes:gradle-versions-plugin:0.28.0"
        }
    }
    复制代码
    ```
    
*   3. 添加完成之后，在根目录下执行以下命令。
    
    ```
    ./gradlew dependencyUpdates
    复制代码
    ```
    
    会在当前目录下生成 build/dependencyUpdates/report.txt 文件。
    

MVVM 架构
-------

Jetpack 实战项目 PokemonGo 基于 MVVM 架构和 Repository 设计模式，如今几乎所有的 Android 开发者至少都听过 MVVM 架构，在谷歌 Android 团队宣布了 Jetpack 的视图模型之后，它已经成为了现代 Android 开发模式最流行的架构之一，如下图所示：

![](simpread-神奇宝贝  眼前一亮的 Jetpack + MVVM 极简实战.assets/2b3137493b5b451fadbca1a435f91e13tplv-k3u1fbpfcp-watermark.awebp)

MVVM 有助于将应用程序的业务逻辑与 UI 完全分开。 如果业务逻辑与 UI 逻辑之间的联系非常紧密，那么维护将很困难，由于很难重用业务逻辑，因此编写单元测试代码非常困难，一堆重复的代码和复杂的逻辑。

Jetpack 的视图模型的 MVVM 架构由 View + DataBinding + ViewModel + Model 组成。

### DataBinding

DataBinding（数据绑定）实际上是 XML 布局中的另一个视图结构层次，视图 (XML) 通过数据绑定层不断地与 ViewModel 交互。

我们来看一个例子，首页上有个 RecyclerView 用来展示神奇宝贝数据（名字、图片、点击事件等等），每一个 item 对应一个 ViewHolder，来看一下 ViewHolder 的实现。

```kotlin
class PokemonViewModel(view: View) : DataBindingViewHolder<PokemonListModel>(view) {
    private val mBinding: RecycleItemPokemonBinding by viewHolderBinding(view)

    override fun bindData(data: PokemonListModel, position: Int) {
        mBinding.apply {
            pokemon = data
            executePendingBindings()
        }
    }

}
复制代码
```

正如你所看到的，由于使用了数据绑定，ViewHolder 里面的代码变的非常简单，可能这个例子不够明显，我们来看一个劲爆的，点击首页每一个 item 会跳转到详情页面，详情页面如下图所示：

![](simpread-神奇宝贝  眼前一亮的 Jetpack + MVVM 极简实战.assets/d9618eedb09948bc9c7425fe0ab8478etplv-k3u1fbpfcp-watermark.awebp)

详情页面（DetailActivity）展示了神奇宝贝的详细数据，先查询数据库，如果没有找到，读取网路数据然后保存到数据库，由于使用了数据绑定，代码变得非常简单，如下所示：

```kotlin
class DetailActivity : DataBindingAppCompatActivity() {

    private val mBindingActivity: ActivityDetailsBinding by binding(R.layout.activity_details)
    private val mViewModel: DetailViewModel by viewModels()
    lateinit var mPokemonModel: PokemonListModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        mBindingActivity.apply {
            mPokemonModel = requireNotNull(intent.getParcelableExtra(KEY_LIST_MODEL))
            pokemonListModel = mPokemonModel
            lifecycleOwner = this@DetailActivity
            viewModel = mViewModel.apply {
                fectchPokemonInfo(mPokemonModel.name)
                    .observe(this@DetailActivity, Observer {})
            }
        }
    }
}
复制代码
```

正如你所见 DetailActivity 代码变得非常简单，如果以后我们想要改变网络的 URL、Model、获取或保存数据的方式等等，我们不需要改变 DetailActivity 中的任何代码。

更多关于 DataBinding 的使用请参考我另外一个仓库 [JDataBinding](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fhi-dhl%2FJDataBinding "https://github.com/hi-dhl/JDataBinding")：目前已经封装了一系列的组件包含 DataBindingActivity、DataBindingAppCompatActivity、DataBindingFragmentActivity、DataBindingFragment、DataBindingDialog、DataBindingListAdapter、DataBindingViewHolder 等等。

### ViewModel

ViewModel 是 MVVM 架构中非常重要的设计，它在 activities 或 fragments 和业务逻辑中起到了非常重要的作用，它不依赖于 UI 组件，使得单元测试更加容易，ViewModel 以生命周期的方式管理界面相关的数据，直到 Activity 被销毁。

LiveData 与 ViewModel 具有很好的协同作用，LiveData 持有从数据源获取到的数据，并且它可以被 DataBinding 组件观察，当 Activity 被销毁时，它将被取消订阅。

而详情页面（DetailActivity) 代码之所以能这么简单得益于 ViewModel、LiveData、DataBinding 协同工作, 我们来看一下 ViewModel 代码。

```
class DetailViewModel @ViewModelInject constructor(
    val polemonRepository: Repository
) : ViewModel() {
    private val _pokemon = MutableLiveData<PokemonInfoModel>()
    val pokemon: LiveData<PokemonInfoModel> = _pokemon

    @OptIn(ExperimentalCoroutinesApi::class)
    fun fectchPokemonInfo(name: String) = liveData<PokemonInfoModel> {
        polemonRepository.featchPokemonInfo(name)
        .collectLatest {
                _pokemon.postValue(it)
                emit(it)
            }
        .......
        // 省略部分代码，
    }

}
复制代码
```

**activity_details.xml 代码**

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable
            
            type="com.hi.dhl.pokemon.ui.detail.DetailViewModel" />

    </data>
    
    ......
    <androidx.appcompat.widget.AppCompatTextView
        android:id="@+id/weight"
        android:text="@{viewModel.pokemon.getWeightString}"/>
    ......
    
</layout>
```

这是获取神奇宝贝的详细信息，通过 DataBinding 以声明方式将数据（神奇宝贝的体重）绑定到界面上，更多使用参考项目中的代码。

### Repository

Repository 设计模式是最流行、应用最广泛的设计模式之一，在 Repository 层获取网络数据，并将数据存储到数据库中，在这一层中有两个非常重要的成员 Paging3 库中的 `RemoteMediator` 和 Data Mappers。

#### RemoteMediator

在之前的文章 [Jetpack 成员 Paging3 实践以及源码分析（一）](https://juejin.cn/post/6844904193468137486 "https://juejin.cn/post/6844904193468137486") 和 [Jetpack 新成员 Paging3 网络实践及原理分析（二）](https://juejin.cn/post/6844904196207345672 "https://juejin.cn/post/6844904196207345672") 分别分析了使用 Paging3 访问 **数据库** 和 **网络**，但是遗漏了 `RemoteMediator` 类的使用，RemoteMediator 是 Paging3 当中一个非常重要的成员，用于实现 **数据库** 和 **网络** 访问，所以这里是对之前的文章一个补充。

> `RemoteMediator` 很重要，需要单独花一篇文章去分析，为了节省篇幅，在这里不会详细的去分析它，如果对 `RemoteMediator` 不太理解没有关系，我会在后续的文章里面详细的分析它。

项目中网络访问用的是 [Retrofit2 & OkHttp3](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fsquare%2Fretrofit "https://github.com/square/retrofit") 用来请求网络数据，使用 Room 作为数据库存储，将获得的数据保存到数据库中，Room 在 SQLite 上提供了一个抽象层，流畅地访问 SQLite 数据库，同时拥有了 SQLite 全部功能，在编译的时候进行错误检查。

```
@OptIn(ExperimentalPagingApi::class)
class PokemonRemoteMediator(
    val api: PokemonService,
    val db: AppDataBase
) : RemoteMediator<Int, PokemonEntity>() {
    val mPageKey = 0
    override suspend fun load(
        loadType: LoadType,
        state: PagingState<Int, PokemonEntity>
    ): MediatorResult {
        try {

            ......
            val pageKey = when (loadType) {
                // 首次访问 或者调用 PagingDataAdapter.refresh()
                LoadType.REFRESH -> null
                // 在当前加载的数据集的开头加载数据时
                LoadType.PREPEND -> return MediatorResult.Success(endOfPaginationReached = true)
                // 在当前数据集末尾添加数据
                LoadType.APPEND -> {
                    ......
                    if (remoteKey == null || remoteKey.nextKey == null) {
                        return MediatorResult.Success(endOfPaginationReached = true)
                    }
                    remoteKey.nextKey
                }
            }

            ......
            // 使用 Retrofit2 获取网络数据
            val page = pageKey ?: 0
            val result = api.fetchPokemonList(
                state.config.pageSize,
                page * state.config.pageSize
            ).results
            

            .......

            db.withTransaction {
                if (loadType == LoadType.REFRESH) { // 当首次加载，或者下拉刷新的时候，清空当前数据 }
                ......
                // 存储获取到的数据
                remoteKeysDao.insertAll(entity)
                pokemonDao.insertPokemon(item)
            }

            return MediatorResult.Success(endOfPaginationReached = endOfPaginationReached)
        } catch (e: IOException) {
            return MediatorResult.Error(e)
        } catch (e: HttpException) {
            return MediatorResult.Error(e)
        }
    }
}
复制代码
```

**注意：使用了 `@OptIn(ExperimentalPagingApi::class)` 需要在 App 模块 build.gradle 文件内添加以下代码。**

```
android {
    kotlinOptions {
        freeCompilerArgs += ["-Xopt-in=kotlin.RequiresOptIn"]
    }
}
复制代码
```

在 `RemoteMediator` 的实现类 `PokemonRemoteMediator` 中的核心部分是关于参数 LoadType 的判断。

*   `LoadType.REFRESH`：**首次访问** 或者调用 **PagingDataAdapter.refresh(**) 触发，这里不需要做任何操作，返回 null 就可以
*   `LoadType.PREPEND`：在当前列表头部添加数据的时候时触发，实际在项目中基本很少会用到直接返回 `MediatorResult.Success(endOfPaginationReached = true)` ，参数 endOfPaginationReached 表示没有数据了不在加载
*   `LoadType.APPEND`：下拉加载更多时触发，这里获取下一页的 key, 如果 key 不存在，表示已经没有更多数据，直接返回 `MediatorResult.Success(endOfPaginationReached = true)` 不会在进行网络和数据库的访问

接下来的逻辑和之前请求网络数据的逻辑没有什么区别了，使用 Retrofit2 获取网络数据，然后使用 Room 将数据保存到数据库中。

接下来聊一下 Repository 中另外一个重要的成员 Data Mapper，在项目中起到了非常的重要，在一个快速开发的项目中，为了越快完成第一个版本交付，下意识的将数据源和 UI 绑定到一起，当业务逐渐增多，数据源变化了，上层也要一起变化，导致后期的重构工作量很大，核心的原因耦合性太强了。

#### Data Mapper（个人建议）

Data Mapper 的意识非常重要，在项目中起到了非常的重要，关于 Data Mappers 在 Repository 中的重要性可以看一下国外大神写的这篇文章 [The “Real” Repository Pattern in Android](https://link.juejin.cn?target=https%3A%2F%2Fproandroiddev.com%2Fthe-real-repository-pattern-in-android-efba8662b754 "https://proandroiddev.com/the-real-repository-pattern-in-android-efba8662b754") 在 Medium 上获得了 4.9K 的赞。

使用 Data Mapper 分离数据源的 Model 和 页面显示的 Model，不要因为数据源的增加、修改或者删除，导致上层页面也要跟着一起修改，换句话说使用 Data Mapper 做一个中间转换，如下图所示，来源于网络：

![](simpread-神奇宝贝  眼前一亮的 Jetpack + MVVM 极简实战.assets/f011dff8132f46a597190af6ff064b22tplv-k3u1fbpfcp-watermark.awebp)

使用 Data Mapper（数据映射）优点如下：

*   数据源的更改不会影响上层的业务
*   糟糕的后端实现不会影响上层的业务 (想象一下，如果你被迫执行 2 个网络请求，因为后端不能在一个请求中提供你需要的所有信息，你会让这个问题影响你的整个代码吗?)
*   Data Mapper 便于做单元测试，确保不会因为数据源的变化，而影响上层的业务

如果在一个大型项目中直接使用 Data Mapper 会有适得其反的效果，所以需要结合设计模式来完善，这不在本文讨论范围之内，其实在这里我想表达是，不要因为快速实现某个功能，下意识的将数据源的 model 和 UI 绑定在一起。

Data Mappe 实现方式有很多种，可以手动实现，也可以通过引入第三方框架，其中有名框架 [modelmapper](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fmodelmapper%2Fmodelmapper "https://github.com/modelmapper/modelmapper")，在 PokemonGo 项目中是手动实现的。

### Kotlin Flow

停止使用 RxJava，尝试一下 Flow，不仅简单而且功能很强大，Retrofit2 和 Room 也都提供了对应的支持。

Flow 库是在 Kotlin Coroutines 1.3.2 发布之后新增的库，也叫做异步流，类似 RxJava 的 Observable，在 PokemonGo 项目中也用到了 Flow。

```
override suspend fun featchPokemonInfo(name: String): Flow<PokemonInfoModel> {
    return flow {
        val pokemonDao = db.pokemonInfoDao()
        var infoModel = pokemonDao.getPokemon(name)
        // 查询数据库是否存在，如果不存在请求网络
        if (infoModel == null) {
            // 网络请求
            val netWorkPokemonInfo = api.fetchPokemonInfo(name)
            ......
            pokemonDao.insertPokemon(infoModel) // 插入更新数据库
        }

        val model = mapper2InfoModel.map(infoModel) // 数据转换
        emit(model)
    }.flowOn(Dispatchers.IO)
}
复制代码
```

在这里做了三件事：

*   查询数据库是否存在，如果不存在请求网络
*   请求网络获取数据，更新数据库
*   将数据源的 Model 转换为页面显示的 Model

### 依赖注入

Hilt、Dagger、Koin 等等都是依赖注入库，使用依赖注入库有以下优点：

*   依赖注入库会自动释放不再使用的对象，减少资源的过度使用。
*   在配置 scopes 范围内，可重用依赖项和创建的实例，提高代码的可重用性，减少了很多模板代码。
*   代码变得更具可读性。
*   易于构建对象。
*   编写低耦合代码，更容易测试。

在 PokemonGo 项目中使用的是 Hilt，Hilt 是在 Dagger 基础上进行开发的，减少了在项目中进行手动依赖，Hilt 集成了 Jetpack 库和 Android 框架类，并删除了大部分模板代码，让开发者只需要关注如何进行绑定，同时 Hilt 也继承了 Dagger 优点，编译时正确性、运行时性能、并且得到了 Android Studio 的支持，来看一下 Hilt 与 Room 在一起使用的例子。

```
@Module
@InstallIn(ApplicationComponent::class)
object RoomModule {

    /**
     * @Provides 常用于被 @Module 注解标记类的内部的方法，并提供依赖项对象。
     * @Singleton 提供单例
     */
    @Provides
    @Singleton
    fun provideAppDataBase(application: Application): AppDataBase {
        return Room
            .databaseBuilder(application, AppDataBase::class.java, "dhl.db")
            .fallbackToDestructiveMigration()
            .allowMainThreadQueries()
            .build()
    }
    
    @Singleton
    @Provides
    fun provideTasksRepository(
        db: AppDataBase
    ): Repository {
        return PokemonFactory.makePokemonRepository(db)
    }
}
复制代码
```

这里需要用到 @Module 注解，使用 @Module 注解的普通类，在其内部提供 Room 的实例，更多使用可以查看 PokemonGo 项目。

### 小巧灵活的进度条

神奇宝贝详情页的进度条使用的是 [JProgressView](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fhi-dhl%2FJProgressView "https://github.com/hi-dhl/JProgressView") ：一个小巧灵活可定制的进度条，支持图形：圆形、圆角矩形、矩形等等，效果如下图所示：

![](simpread-神奇宝贝  眼前一亮的 Jetpack + MVVM 极简实战.assets/d818576e913146a3b2b2d0df34ddc222tplv-k3u1fbpfcp-watermark-16329227162745.awebp)

起源于当时想用一个现成的库，但是在网上找了很多，没有一个合适自己的，要不大而全，要不作者好久没更新了，要不不兼容 DataBinding，于是乎就自己封装了一个小巧灵活的进度条，项目长期维护并持续更新，如果有更好的建议欢迎告知我，JProgressView 使用非常的简单，根据自己的需求去配置即可。

```
<com.hi.dhl.jprogressview.JProgressView
    android:layout_width="match_parent"
    android:layout_height="18dp"
    android:layout_below="@+id/exp"
    android:translationZ="100dp"
    app:maxProgressValue="@{viewModel.pokemon.maxExp}"
    app:progressValue="@{viewModel.pokemon.exp}"
    app:progress_animate_duration="@integer/progress_animate_duration"
    app:progress_color="@color/color_progress_4"
    app:progress_color_background="@color/color_progress_bg"
    app:progress_paint_bg_width="@dimen/circle_stroke_width"
    app:progress_paint_value_width="@dimen/circle_stroke_width"
    app:progress_text_color="@android:color/black"
    app:progress_text_size="@dimen/text_size_12sp"
    app:progress_type="@integer/porgress_tpye_round_rect" />
复制代码
```

<table><thead><tr><th>名称</th><th>值类型</th><th>默认值</th><th>备注</th></tr></thead><tbody><tr><td>progress_type</td><td>integer</td><td>圆形：1</td><td>矩形：0；矩形：0；矩形：0</td></tr><tr><td>progress_animate_duration</td><td>integer</td><td>2000</td><td>动画运行时间</td></tr><tr><td>progress_color</td><td>color</td><td>Color.GRAY</td><td>当前进度颜色</td></tr><tr><td>progress_color_background</td><td>color</td><td>Color.GRAY</td><td>进度条背景颜色</td></tr><tr><td>progress_paint_bg_width</td><td>dimen</td><td>10</td><td>进度条背景画笔的宽度</td></tr><tr><td>progress_paint_value_width</td><td>dimen</td><td>10</td><td>当前进度画笔的宽度</td></tr><tr><td>progress_text_color</td><td>color</td><td>Color.BLUE</td><td>进度条上的文字的颜色</td></tr><tr><td>progress_text_size</td><td>dimen</td><td><code>sp2Px(20f)</code></td><td>进度条上的文字的大小</td></tr><tr><td>progress_text_visible</td><td>boolean</td><td>默认不显示：false</td><td>是否显示文字</td></tr><tr><td>progress_value</td><td>integer</td><td>0</td><td>当前进度</td></tr><tr><td>progress_value_max</td><td>integer</td><td>100</td><td>当前进度条的最大值</td></tr></tbody></table>

更多关于进度条的使用，查看 [JProgressView](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fhi-dhl%2FJProgressView "https://github.com/hi-dhl/JProgressView") 仓库，全文到这里就结束了，为了节省篇幅，很多在之前系列文章里面分析过的，这里不在详细分析了，更多技术细节会在后续的系列文章中分析。

> 正在建立一个最全、最新的 AndroidX Jetpack 相关组件的实战项目 以及 相关组件原理分析文章，目前已经包含了 App Startup、Paging3、Hilt 等等，正在逐渐增加其他 Jetpack 新成员，仓库持续更新，可以前去查看：[AndroidX-Jetpack-Practice](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fhi-dhl%2FAndroidX-Jetpack-Practice "https://github.com/hi-dhl/AndroidX-Jetpack-Practice"), 如果这个仓库对你有帮助，请仓库右上角帮我点个赞。

结语
--

致力于分享一系列 Android 系统源码、逆向分析、算法、翻译、Jetpack 源码相关的文章，正在努力写出更好的文章，如果这篇文章对你有帮助给个 star，文章中有什么没有写明白的地方，或者有什么更好的建议欢迎留言，欢迎一起来学习，在技术的道路上一起前进。

### 算法

由于 LeetCode 的题库庞大，每个分类都能筛选出数百道题，由于每个人的精力有限，不可能刷完所有题目，因此我按照经典类型题目去分类、和题目的难易程度去排序。

*   数据结构： 数组、栈、队列、字符串、链表、树……
*   算法： 查找算法、搜索算法、位运算、排序、数学、……

每道题目都会用 Java 和 kotlin 去实现，并且每道题目都有解题思路、时间复杂度和空间复杂度，如果你同我一样喜欢算法、LeetCode，可以关注我 GitHub 上的 LeetCode 题解：[Leetcode-Solutions-with-Java-And-Kotlin](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fhi-dhl%2FLeetcode-Solutions-with-Java-And-Kotlin "https://github.com/hi-dhl/Leetcode-Solutions-with-Java-And-Kotlin")，一起来学习，期待与你一起成长。

### Android 10 源码系列

正在写一系列的 Android 10 源码分析的文章，了解系统源码，不仅有助于分析问题，在面试过程中，对我们也是非常有帮助的，如果你同我一样喜欢研究 Android 源码，可以关注我 GitHub 上的 [Android10-Source-Analysis](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fhi-dhl%2FAndroid10-Source-Analysis "https://github.com/hi-dhl/Android10-Source-Analysis")，文章都会同步到这个仓库。

*   [0xA01 Android 10 源码分析：APK 是如何生成的](https://juejin.cn/post/6844904063155322887 "https://juejin.cn/post/6844904063155322887")
*   [0xA02 Android 10 源码分析：APK 的安装流程](https://juejin.cn/post/6844904078015725576 "https://juejin.cn/post/6844904078015725576")
*   [0xA03 Android 10 源码分析：APK 加载流程之资源加载](https://juejin.cn/post/6844904089696862221 "https://juejin.cn/post/6844904089696862221")
*   [0xA04 Android 10 源码分析：APK 加载流程之资源加载（二）](https://juejin.cn/post/6844904105433907207 "https://juejin.cn/post/6844904105433907207")
*   [0xA05 Android 10 源码分析：Dialog 加载绘制流程以及在 Kotlin、DataBinding 中的使用](https://juejin.cn/post/6844904122467123214 "https://juejin.cn/post/6844904122467123214")
*   [0xA06 Android 10 源码分析：WindowManager 视图绑定以及体系结构](https://juejin.cn/post/6844904146227691527 "https://juejin.cn/post/6844904146227691527")
*   [0xA07 Android 10 源码分析：Window 的类型 以及 三维视图层级分析](https://juejin.cn/post/6844904180306411528 "https://juejin.cn/post/6844904180306411528")
*   [更多......](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fhi-dhl%2FAndroid10-Source-Analysis "https://github.com/hi-dhl/Android10-Source-Analysis")

### Android 应用系列

*   [如何在项目中封装 Kotlin + Android Databinding](https://juejin.cn/post/6844904131803480071 "https://juejin.cn/post/6844904131803480071")
*   [再见吧 buildSrc, 拥抱 Composing builds 提升 Android 编译速度](https://juejin.cn/post/6844904176250519565 "https://juejin.cn/post/6844904176250519565")
*   [为数不多的人知道的 Kotlin 技巧以及 原理解析](https://juejin.cn/post/6844904184974835720 "https://juejin.cn/post/6844904184974835720")
*   [Jetpack 最新成员 AndroidX App Startup 实践以及原理分析](https://juejin.cn/post/6844904190440013837 "https://juejin.cn/post/6844904190440013837")
*   [Jetpack 成员 Paging3 实践以及源码分析（一）](https://juejin.cn/post/6844904193468137486 "https://juejin.cn/post/6844904193468137486")
*   [Jetpack 新成员 Paging3 网络实践及原理分析（二）](https://juejin.cn/post/6844904196207345672 "https://juejin.cn/post/6844904196207345672")
*   [Jetpack 新成员 Hilt 实践（一）启程过坑记](https://juejin.cn/post/6844904198803292173?utm_source=gold_browser_extension "https://juejin.cn/post/6844904198803292173?utm_source=gold_browser_extension")
*   [Jetpack 新成员 Hilt 实践之 App Startup（二）进阶篇](https://juejin.cn/post/6844904200590065672 "https://juejin.cn/post/6844904200590065672")
*   [Jetpack 新成员 Hilt 与 Dagger 大不同（三）落地篇](https://juejin.cn/post/6845166890562617352 "https://juejin.cn/post/6845166890562617352")
*   [全方面分析 Hilt 和 Koin 性能](https://juejin.cn/post/6846687596370722823 "https://juejin.cn/post/6846687596370722823")

### 精选译文

目前正在整理和翻译一系列精选国外的技术文章，不仅仅是翻译，很多优秀的英文技术文章提供了很好思路和方法，每篇文章都会有**译者思考**部分，对原文的更加深入的解读，可以关注我 GitHub 上的 [Technical-Article-Translation](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fhi-dhl%2FTechnical-Article-Translation "https://github.com/hi-dhl/Technical-Article-Translation")，文章都会同步到这个仓库。

*   [[译][Google 工程师] 刚刚发布了 Fragment 的新特性 “Fragment 间传递数据的新方式” 以及源码分析](https://juejin.cn/post/6844904151508320263 "https://juejin.cn/post/6844904151508320263")
*   [[译][Google 工程师] 详解 FragmentFactory 如何优雅使用 Koin 以及部分源码分析](https://juejin.cn/post/6844904167685750798 "https://juejin.cn/post/6844904167685750798")
*   [[译][2.4K Start] 放弃 Dagger 拥抱 Koin](https://juejin.cn/post/6844904158324064269?utm_source=gold_browser_extension "https://juejin.cn/post/6844904158324064269?utm_source=gold_browser_extension")
*   [[译][5k+] Kotlin 的性能优化那些事](https://juejin.cn/post/6844904161419640840#heading-7 "https://juejin.cn/post/6844904161419640840#heading-7")
*   [[译] 解密 RxJava 的异常处理机制](https://juejin.cn/post/6844904168583331854 "https://juejin.cn/post/6844904168583331854")
*   [[译][1.4K+ Star] Kotlin 新秀 Coil VS Glide and Picasso](https://juejin.cn/post/6844904182781050893 "https://juejin.cn/post/6844904182781050893")
*   [更多......](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fhi-dhl%2FTechnical-Article-Translation "https://github.com/hi-dhl/Technical-Article-Translation")

### 工具系列

*   [为数不多的人知道的 AndroidStudio 快捷键 (一)](https://juejin.cn/post/6844904020511981581 "https://juejin.cn/post/6844904020511981581")
*   [为数不多的人知道的 AndroidStudio 快捷键 (二)](https://juejin.cn/post/6844904023082926087 "https://juejin.cn/post/6844904023082926087")
*   [关于 adb 命令你所需要知道的](https://juejin.cn/post/6844903918112079885 "https://juejin.cn/post/6844903918112079885")
*   [10 分钟入门 Shell 脚本编程](https://juejin.cn/post/6844903553119748109 "https://juejin.cn/post/6844903553119748109")
*   [基于 Smali 文件 Android Studio 动态调试 APP](https://juejin.cn/post/6844903798972891144 "https://juejin.cn/post/6844903798972891144")
*   [解决在 Android Studio 3.2 找不到 Android Device Monitor 工具](https://juejin.cn/post/6844903773890936839 "https://juejin.cn/post/6844903773890936839")