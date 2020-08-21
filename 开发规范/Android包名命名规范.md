# 简介

Android 开发中规范的包名，有益于程序的开发和阅读， 会直接影响到整个app工程开发效率和扩展性。



# 包的命名

采用反域名命名规则，全部使用**小写字母**, 如：com.leiman.cloudgame形式

* 一级包名为： com
* 二级包名：为个人或公司名称，可以简写
* 三级包名：根据应用（或者组件）进行命名
* 四级包名：模块名或层级名
* 根据实际情况也是可以用五级包名，六级包名。



# 目录命名

android包的目录命名的方式有两种：PBL和PBF。

## PBL（Package By Layer）

字面意思是按层次 ，实际上是按职能划分。比如app里面分为activity、fragment、view、service、db、net、util、base等。

activity的职能是管理所有的activity类，只要这个类是extends Activity的，都放到这个目录下面；以此类推，fragment目录放所有extends Fragment的类。view放置自定义的View，net放置管理网络相关的，bean放置
bean对象。

参考如下目录结构：

```java
└—com
  └—android
    └—demo
      ├—activity
      │ LoginActivity.java
      │ SettingActivity.java
      │
      ├—bean
      │ UserBean.java
      │
      ├—db
      │ DBManager.java
      │
      ├—fragment
      │ NewsFragment.java
      │ PictureFragment.java
      │
      ├—net
      │ RetrofitManager.java
      │
      └—service
      │ BackgroundService.java
```



### 优点

项目结构简洁明了，上手快。适合业务简单，开发人员不多，后期变动不大的项目。

### 缺点

* 低内聚
  同一个包下会是各种功能模块的集合。比如activity包，下面放置了登录、设置等功能模块；这几个模块本身并没有很强的关联性，却放在了一起，聚合性降低。
  
* 高耦合
  这里讲的高耦合是指package之间的关联性，比如activity包内的类往往引用到了fragment或者view里面的类，package之间的关联性比较紧密。
  
* 开发效率
  开发一个功能模块，往往需要到不同的package里面来回切换。比如登录模块，需要到activity里面开发LoginActivity，Activity往往包含了fragment，这时又需要去fragment里面找到对应的fragment来开发。
  
  同样，修改、调试一个功能也需要这样的操作。如果后期项目功能和代码增多，会大大降低开发效率。

## PBF(Package By Feature)

按照功能划分包名。app里面有什么功能模块，就以这个功能模块作为包名，所有这个功能模块的开发都在这个包名下进行。

```java
└─com
    └─google
        └─samples
            └─apps
                └─iosched
                    │  AppApplication.java  定义Application类
                    │  Config.java          定义配置数据（常量）
                    │
                    ├─about
                    │      AboutActivity.java
                    │
                    ├─appwidget
                    │      ScheduleWidgetProvider.java
                    │      ScheduleWidgetRemoteViewsService.java
                    │
                    ├─feedback
                    │      FeedbackApiHelper.java
                    │      FeedbackConstants.java
                    │      FeedbackHelper.java
                    │      ...
                    ├─map
                    │  │  InlineInfoFragment.java
                    │  │  MapActivity.java
                    │  │  MapFragment.java
                    │  │  ...
                    │  │
                    │  └─util
                    │          CachedTileProvider.java
                    │          MarkerLoadingTask.java
                    │          MarkerModel.java
                    │          ...
                    │
                    ├─model
                    │      ScheduleHelper.java
                    │      ScheduleItem.java
                    │      ScheduleItemHelper.java
                    │      ...定义model以及实现model相关操作
                    │
                    ├─provider
                    │      ScheduleContract.java
                    │      ScheduleContractHelper.java
                    │      ScheduleDatabase.java
                    │      ...实现ContentProvider
                    │      （也在此处定义provider依赖的其它类，比如db操作）
                    │
                    ├─receiver
                    │      SessionAlarmReceiver.java
                    │
                    ├─service
                    │      DataBootstrapService.java
                    │      SessionAlarmService.java
                    │      SessionCalendarService.java
                    │
                    ├─session
                    │      SessionDetailActivity.java
                    │      SessionDetailConstants.java
                    │      SessionDetailFragment.java
                    │      ...
                    │
                    ├─settings
                    │      ConfMessageCardUtils.java
                    │      SettingsActivity.java
                    │      SettingsUtils.java
                    │
                    ├─social
                    │      SocialActivity.java
                    │      SocialFragment.java
                    │      SocialModel.java
                    ├─ui
                    │  │  BaseActivity.java
                    │  │  CheckableLinearLayout.java
                    │  │  SearchActivity.java
                    │  │  ...BaseActivity以及自定义UI组件
                    │  │
                    │  └─widget
                    │          AspectRatioView.java
                    │          BakedBezierInterpolator.java
                    │          BezelImageView.java
                    │          ...自定义小UI控件
                    │
                    ├─util
                    │      AboutUtils.java
                    │      AccountUtils.java
                    │      AnalyticsHelper.java
                    │      ...工具类，提供静态方法
```



### 优点

* 高内聚

  所有功能都在一个包名下面完成，所有的activity和fragment都在这里面，包括相关的util。比如map模块：

```java
                      ├─map
                      │  │  InlineInfoFragment.java
                      │  │  MapActivity.java
                      │  │  MapFragment.java
                      │  │  ...
                      │  │
                      │  └─util
                      │          CachedTileProvider.java
                      │          MarkerLoadingTask.java
                      │          MarkerModel.java
                      │          ...
                      │
```

  **说明**：功能模块里面的util一般都是和这个功能模块强相关的，如果是功能模块包名外的util包名，一般放置的是跟项目相关的util类，能作用于整个或者多个功能模块的util类。



* 低耦合

  package之间没有很强的关联性，开发此模块的功能只需要在对应的包名下面进行开发即可，除了基础类外，一般不需要引入其它包的类。

  开发效率高，增删改查都只要在对应的包名下面进行即可，后面的开发人员接手也方便，便于后期组件化转化。

  

* package的大小有意义了

  PBL中包的大小无限增长是合理的，因为功能越添越多

  而PBF中包太大（包里class太多）表示这块需要重构（划分子包）



# 组件化介绍

## 简介

* 什么是组件，什么是组件化？

  **组件（Component）**是对数据和方法的简单封装，功能单一，高内聚，并且是业务能划分的最小粒度。

  **组件化** 就是基于组件可重用的目的上，将一个大的软件系统按照分离关注点的形式，拆分成多个独立的组件。



* 组件化、模块化容易混淆，两者区别又是什么？

  **模块化更加侧重于业务功能的划分，偏向于复用，组件化更加侧重于单一功能的内聚，偏向于解耦。**

  模块化就是将一个程序**按照其功能做拆分，分成相互独立的模块**，以便于每个模块只包含与其功能相关的内容，模块我们相对熟悉,比如登录功能可以是一个模块,搜索功能可以是一个模块等等。

  

  组件化就是**更关注可复用性，更注重关注点分离**，如果从集合角度来看的话，可以说往往一个模块包含了一个或多个组件，或者说模块是一个容器，由组件组装而成。简单来说，组件化相比模块化粒度更小，两者的本质思想都是一致的，都是把大往小的方向拆分，都是为了复用和解耦。



* 组件化好处
  * 代码简洁，冗余量少，维护方便，易扩展新功能。
  * 提高编译速度，从而提高并行开发效率。
  * 避免模块之间的交叉依赖，做到低耦合、高内聚。
  * 引用的第三方库代码统一管理，避免版本统一，减少引入冗余库。
  * 定制项目可按需加载，组件之间可以灵活组建，快速生成不同类型的定制产品。
  * 制定相应的组件开发规范，可促成代码风格规范，写法统一。
  * 系统级的控制力度细化到组件级的控制力度，复杂系统构建变成组件构建。
  * 每个组件有自己独立的版本，可以独立编译、测试、打包和部署。



## 架构模式

* APP 单一工程模式

![APP单一工程模式架构](https://images.xiaozhuanlan.com/photo/2018/d52087abfdcd5c0aaa99c1500991d161.png)



整个项目中也没有模块的概念，只有简单的以业务逻辑划分的文件夹，并且业务之间也是直接相互调用、高度耦合在一起的

![业务模块互相耦合](https://images.xiaozhuanlan.com/photo/2018/00332fcac40f3e4afbba6849a42b4ded.png)



* App 组件化工程模式

![A组件化框架架构](https://images.xiaozhuanlan.com/photo/2018/808208c112b038329f5f69b7dc4a308c.png)

**业务组件之间是独立的，互相没有关联**，这些业务组件在集成模式下是一个个 Library，被 APP 壳工程所依赖，组成一个具有完整业务功能的 APP 应用，但是在组件开发模式下，业务组件又变成了一个个 Application，它们可以独立开发和调试，由于在组件开发模式下，业务组件们的代码量相比于完整的项目差了很远，因此在运行时可以显著减少编译时间。

各个业务组件通信是通过路由转发，如图：

![路由转发通信](https://images.xiaozhuanlan.com/photo/2018/e04fe8b29e0b844855c92f8d56cea527.png)



## 存在问题

1. 代码解耦问题

对已存在的项目进行模块拆分，模块分为两种类型，一种是功能组件模块，封装一些公共的方法服务等，作为依赖库对外提供，一种是业务组件模块，专门处理业务逻辑等功能，这些业务组件模块最终负责组装APP。

2. 组件单独运行问题

通过 **Gradle** 脚本配置方式，进行不同环境切换。比如只需要把 Apply plugin: 'com.android.library' 切换成Apply plugin: 'com.android.application' 就可以，同时还需要在 AndroidManifest 清单文件上进行设置，因为一个单独调试需要有一个入口的 Activity。比如设置一个变量 isModule，标记当前是否需要单独调试，根据isModule 的取值，使用不同的 gradle 插件和 AndroidManifest 清单文件，甚至可以添加 Application 等 Java 文件，以便可以做一下初始化的操作。

3. 组件间通信问题

通过接口+实现的结构进行组件间的通信。每个组件声明自己提供的服务 Service API，这些 Service 都是一些接口，组件负责将这些 Service 实现并注册到一个统一的路由 Router 中去，如果要使用某个组件的功能，只需要向Router 请求这个 Service 的实现，具体的实现细节我们全然不关心，只要能返回我们需要的结果就可以了。在组件化架构设计图中 Common 组件就包含了路由服务组件，里面包括了每个组件的路由入口和跳转。

4. UI 跳转问题

可以说 UI 跳转也是组件间通信的一种，但是属于比较特殊的数据传递。不过一般 UI 跳转基本都会单独处理，一般通过短链的方式来跳转到具体的 Activity。每个组件可以注册自己所能处理的短链的 Scheme 和 Host，并定义传输数据的格式，然后注册到统一的 UIRouter 中，UIRouter 通过 Scheme 和 Host 的匹配关系负责分发路由。但目前比较主流的做法是通过在每个 Activity 上添加注解，然后通过 APT 形成具体的逻辑代码。目前方式是引用阿里的 [ARouter](https://github.com/alibaba/ARouter) 框架，通过注解方式进行页面跳转。

5. 组件生命周期问题

在架构图中的核心管理组件会定义一个组件生命周期接口，通过在每个组件设置一个配置文件,这个配置文件是通过使用注解方式在编译时自动生成，配置文件中指明具体实现组件生命周期接口的实现类，来完成组件一些需要初始化操作并且做到自动注册，暂时没有提供手动注册的方式。

6. 集成调试问题

每个组件单独调试通过并不意味着集成在一起没有问题，因此在开发后期我们需要把几个组件机集成到一个 APP 里面去验证。由于经过前面几个步骤保证了组件之间的隔离，所以可以任意选择几个组件参与集成，这种按需索取的加载机制可以保证在集成调试中有很大的灵活性，并且可以加大的加快编译速度。需要注意的一点是，每个组件开发完成之后，需要把 isModule 设置为 true并同步，这样主项目就可以通过参数配置统一进行编译。

7. 代码隔离问题

如果还是 implementation project(xxx:xxx.aar) 来引入组件，我们就完全可以直接使用到其中的实现类，那么主项目和组件之间的耦合就没有消除，那之前针对接口编程就变得毫无意义。我们希望只在 assembleDebug 或者 assembleRelease 的时候把 AAR 引入进来，而在开发阶段，所有组件都是看不到的，这样就从根本上杜绝了引用实现类的问题。

目前做法是主项目只依赖 Common 的依赖库，业务组件通过路由服务依赖库按需进行查找，用反射方式进行组件加载，然后在主工程中调用组件服务，组件与组件之间调用则是通过接口+实现进行通信，后续规划通过自定义Gradle 插件，通过字节码自动插入组件的依赖进行编译打包，实现自动筛选 assembleDebug 或 assembleRelease 这两个编译命任务，只有属于包含这两个任务的命令才引入具体实现类，其他的则不引入。

## 具体实践

参考：

[Android组件化开发实践和案例分享](https://juejin.im/post/6844903765565243400) 
[Android 组件化框架设计与实践](https://xiaozhuanlan.com/topic/9850736142) 

# 组件化命名

## Support 和 AndroidX参考

### 常用依赖库对比

详细变化内容，可以下载CSV格式映射文件：[androidx-class-mapping.csv](https://developer.android.google.cn/topic/libraries/support-library/downloads/androidx-class-mapping.csv)

| Old build artifact                                       | AndroidX build artifact                            |
| -------------------------------------------------------- | -------------------------------------------------- |
| `com.android.support:appcompat-v7:28.0.2`                | `androidx.appcompat:appcompat:1.0.0`               |
| `com.android.support:design:28.0.2`                      | `com.google.android.material:material:1.0.0`       |
| `com.android.support:support-v4:28.0.2`                  | `androidx.legacy:legacy-support-v4:1.0.0`          |
| `com.android.support:recyclerview-v7:28.0.2`             | `androidx.recyclerview:recyclerview:1.0.0`         |
| `com.android.support.constraint:constraint-layout:1.1.2` | `androidx.constraintlayout:constraintlayout:1.1.2` |

### 常用支持库类对比

详细变化内容，可以下载CSV格式映射文件：[androidx-artifact-mapping.csv](https://developer.android.google.cn/topic/libraries/support-library/downloads/androidx-artifact-mapping.csv)

| Support Library class                      | AndroidX class                              |
| ------------------------------------------ | ------------------------------------------- |
| `android.support.v4.app.Fragment`          | `androidx.fragment.app.Fragment`            |
| `android.support.v4.app.FragmentActivity`  | `androidx.fragment.app.FragmentActivity`    |
| `android.support.v7.app.AppCompatActivity` | `androidx.appcompat.app.AppCompatActivity`  |
| `android.support.v7.app.ActionBar`         | `androidx.appcompat.app.ActionBar`          |
| `android.support.v7.widget.RecyclerView`   | `androidx.recyclerview.widget.RecyclerView` |

规则大概就是：android.support.xxx 和 androidx.xxx.xxx



## 命名规则[建议]

* 组件module命名

  module +  “-”  + 组件名称（或简写）

  例如：分享组件 module-share

* 通用组件包名

  com.leiman.widget + 组件名称（或简写）

* 业务组件包名

  applicationId  + 组件名称（或简写）

* 资源命名

  所有的资源必须以“{组件名}_{资源名}”命名，防止资源ID冲突。

  例如modulex_activity_home

  

参考：

[Android组件化开发规范](https://juejin.im/post/6844903708908601352#heading-2)

[微信Android模块化架构重构实践](https://mp.weixin.qq.com/s/6Q818XA5FaHd7jJMFBG60w)





