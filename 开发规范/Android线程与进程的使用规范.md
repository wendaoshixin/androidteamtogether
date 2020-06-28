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



**3、【推荐】线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 说明： Executors 返回的线程池对象的弊端如下：**

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


**4、【强制】不要在非 UI 线程中初始化 ViewStub，否则会返回 null。**



**5、【推荐】新建线程时，定义能识别自己业务的线程名称，便于性能优化和问题排查。**
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



**6、【推荐】谨慎使用 Android 的多进程，多进程虽然能够降低主进程的内存压力，但会遇到如下问题：**

1. 不能实现完全退出所有 Activity 的功能；
2. 首次进入新启动进程的页面时会有延时的现象（有可能黑屏、白屏几秒，是白屏还是黑屏和新 Activity 的主题有关）；
3. 应用内多进程时，Application 实例化多次，需要考虑各个模块是否都需要在所有进程中初始化；
4. 多进程间通过 SharedPreferences 共享数据时不稳定。