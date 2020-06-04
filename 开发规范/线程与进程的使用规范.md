# 规范说明

- 【强制】必须遵守，违反本约定或将会引起严重的后果；
- 【推荐】尽量遵守，长期遵守有助于系统稳定性和合作效率的提升；
- 【参考】充分理解，技术意识的引导，是个人学习、团队沟通、项目合作的方向。

# 线程与进程使用

## 线程使用方式

* Thread
* HandleThread和Handler
* ThreadPoolExecutor
* AsyncTask
* View.post(Runable)与runOnUiThread

## 进程通信方式


* 通过Intent

> 四大组件中三大组件Activity、Service、Receiver都支持在Intent中传递Bundle数据。

* 使用AIDL

> AIDL是 Android Interface definition language的缩写，它是一种android内部进程通信接口的描述语言。AIDL可以处理发送到服务器端大量的并发请求（不同与Messenger的串行处理方式），也可以实现跨进程的方法调用。

* 使用Messenger

>Messenger可以理解为信使，通过它可以再不同进程中传递Message对象，在Message中放入我们需要传递的数据，就可以实现数据的进程间传递了。由于它一次处理一个请求，因此在服务端不需要考虑线程同步的问题，因为服务端不存在并发执行的情形。

* 使用ContentProvider

>ContentProvider是Android中提供的专门用于不同应用间进行数据共享的方式，天生适合进程间通信。
>ContentProvider对底层的数据存储方式没有任何要求，既可以使用Sqlite数据库，也可以使用文件方式，甚至可以使用内存中的一个对象来存储。

* 使用Socket

> Socket套接字，是网络通信中的概念，分为流式套接字和用户数据奥套接字两种，对应于网络的传输控制层中的TCP和UDP协议。
> 两个进程可以通过Socket来实现信息的传输，Socket本身可以支持传输任意字节流。

* 共享文件/内存

> 两个进程通过读写同一个文件来交换数据，比如A进程把数据写入文件，B进程通过读取这个文件来获取数据。



**进程间通信方式对比**

| 名称            | 优点                               | 缺点                                         | 试用场景                     |
| --------------- | ---------------------------------- | -------------------------------------------- | ---------------------------- |
| Intent          | 简单易用                           | 只能传输Bundle所支持的数据类型               | 四大组件间的进程通信         |
| 文件/内存共享   | 简单易用                           | 不适合高并发                                 | 简单的数据共享，无高并发场景 |
| AIDL            | 功能强大，支持 一对多并发实时通信  | 稍微复杂，需要注意线程同步                   | 复杂的进程调用               |
| Messenger       | 比AIDL稍微简单易用                 | 比AIDL功能弱，只支持一对多串行实时通信       | 简单进程通信                 |
| ContentProvider | 强大的数据共享功能，可扩展call方法 | 受约束的AIDL，主要对外提供数据的增删查改操作 | 进程间的大量数据共享         |
| Socket          | 跨主机，通信范围广                 | 只能传输原始的字节流                         | 网络通信                     |



# 线程和进程规范

**1、【强制】不要通过 Intent 在Android基础组件之间传递大数据（binder transaction缓存为 2MB），可能导致OOM。**



**2、【强制】在 Application 的业务初始化代码加入进程判断，确保只在自己需要的进程初始化。特别是后台进程减少不必要的业务初始化。**

正例：

```java
public class MyApplication extends Application {
	@Override
	public void onCreate() {
		//在所有进程中初始化
		....
		//仅在主进程中初始化
		if (mainProcess) {
			...
		}
		//仅在后台进程中初始化
		if (bgProcess) {
			...
		}
	}
}

```



**3、【强制】线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 说明： Executors 返回的线程池对象的弊端如下：**

1. FixedThreadPool 和 SingleThreadPool ： 允许的请求队列长度为Integer.MAX_VALUE，可能会堆积大量的请求，从而导致 OOM；
2. CachedThreadPool 和 ScheduledThreadPool ： 允许的创建线程数量为Integer.MAX_VALUE，可能会创建大量的线程，从而导致 OOM。

正例：

```
int NUMBER_OF_CORES = Runtime.getRuntime().availableProcessors();
int KEEP_ALIVE_TIME = 1;
TimeUnit KEEP_ALIVE_TIME_UNIT = TimeUnit.SECONDS;

BlockingQueue<Runnable> taskQueue = new LinkedBlockingQueue<Runnable>();

ExecutorService executorService = new ThreadPoolExecutor(
	NUMBER_OF_CORES, //corePoolSize
	NUMBER_OF_CORES*2, //maximumPoolSize
	KEEP_ALIVE_TIME, //keepAliveTime
	KEEP_ALIVE_TIME_UNIT, //Unit
	taskQueue, //workQueue
	new BackgroundThreadFactory(), //threadFactory
	new DefaultRejectedExecutionHandler() //handler);
复制代码
```

反例：

```
ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
```



**4、【强制】子线程中不能更新界面，更新界面必须在主线程中进行，网络操作不能在主线程中调用。**



**5、【强制】不要在非 UI 线程中初始化 ViewStub，否则会返回 null。**



**6、【推荐】尽量减少不同 APP 之间的进程间通信及拉起行为。拉起导致占用系统资源，影响用户体验。**



**7、【推荐】新建线程时，定义能识别自己业务的线程名称，便于性能优化和问题排查。**
 正例：

```
public class MyThread extends Thread {
	public MyThread(){
		super.setName("ThreadName");
		…
	}
}
复制代码
```



**8、【推荐】ThreadPoolExecutor 设置线程存活时间(setKeepAliveTime)，确保空闲时线程能被释放。**



**9、【推荐】 禁止在多进 程之间用 SharedPreferences 共享数据 ， 虽然可以(MODE_MULTI_PROCESS)，但官方已不推荐。**



**10、【推荐】谨慎使用 Android 的多进程，多进程虽然能够降低主进程的内存压力，但会遇到如下问题：**

1. 不能实现完全退出所有 Activity 的功能；
2. 首次进入新启动进程的页面时会有延时的现象（有可能黑屏、白屏几秒，是白屏还是黑屏和新 Activity 的主题有关）；
3. 应用内多进程时，Application 实例化多次，需要考虑各个模块是否都需要在所有进程中初始化；
4. 多进程间通过 SharedPreferences 共享数据时不稳定。