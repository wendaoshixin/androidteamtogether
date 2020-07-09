## App内存泄漏检测

### 为什么要做内存检测，优化？

​		因为开发的不规范，不顾组件生命周期随意引用，产生了内存泄漏，导致App占用的系统内存不断的增大，终会在某一刻产生让用户抓狂的OOM(Out Of Memory)异常，所以一个好的App，对于内存使用的检测和优化是必不可少的。

### 怎么检测内存泄漏？

Java中存在四种引用类型：

强引用：我们每天写的代码，给对象赋值时，就是强引用关系。内存回收时，系统宁愿抛出OOM，也绝不会回收被强引用持有的对象。				

```java
Object object = new Object();
```

软引用：使用SoftReference<T>修饰要引用的对象。只要内存足够，我们不对对象回收，但是当内存不足， gc对软引用对象进行回收 可以看出软引用对内存很敏感，可用来高速缓存(例如Glide的三级缓存)，同时它可以结合队列使用，如果软引用被gc回收，jvm就会把软引用加入到队列中。

```java
Object object = new Object();
SoftReference<Object> softReference = new SoftReference<Object>(object);
```

弱引用：使用WeakReference<T>修饰要引用的对象。内存回收时，检测到对象只存在弱引用关系时，会立即回收该对象。同时它可以结合队列使用，如果弱引用对象被gc回收，jvm就会把弱引用对象加入到队列中

```java
Object weakObj = new Object();
final WeakReference<Object> weakReference = new WeakReference<>(weakObj);
```

虚引用：虚引用PhantomReference<T>必须和引用队列ReferenceQueue<T>关联使用。被虚饮用修饰的对象，则跟没有任何引用与之关联一样，在任何时候都可能被垃圾回收器回收。当虚引用对象被回收时，gc会发出一个通知，程序可以在此时对该对象做某些操作。

```java
Object phantomObj = new Object();
ReferenceQueue<Object> queue = new ReferenceQueue<>();
final PhantomReference<Object> phantomReference = new PhantomReference<>(phantomObj, queue);
```

<img src="/Users/lm162/Library/Application Support/typora-user-images/image-20200629173707949.png" alt="image-20200629173707949" style="zoom:80%;" />



综合分析下来，强引用首先排除，因为内存泄漏就是强引用关系导致的，软引用需要构造内存不足的情况，才能触发gc回收，实现偏麻烦，而且实际运用中意义不大，虚引用随时可能被回收，不利于观察，所以比较好的实现方式，只有弱引用配合引用队列来实现内存泄漏的检测。以Activity举例，

1 : Activity onCreate时，用一个弱引用去关联该Activity，并配合引用队列观察该弱引用对象。

2 : 在Activity执行onDestory时，我们手动执行一下gc(注意，System.gc在Android 5.0后不会执行内存回收了，见下图源码)

3: 检测与弱引用关联的引用队列中是否包含持有该Activity的弱引用对象，有的话，证明该Activity回收了



Android4.4  System.gc()源码

```java
/**
 * Indicates to the VM that it would be a good time to run the
 * garbage collector. Note that this is a hint only. There is no guarantee
 * that the garbage collector will actually be run.
 */
public static void gc() {
    Runtime.getRuntime().gc();
}
```



Android5.0之后 System.gc()源码，可以看到只有当justRanFinalization为true时，才会去调用。Runtime.getRuntime().gc()，真正起到回收内存的作用，所以5.0之后，可以先调用System.runFinalization()再调用System.gc()，或者直接调用Runtime.getRuntime().gc()

```java
public static void gc() {
    boolean shouldRunGC;
    synchronized (LOCK) {
        shouldRunGC = justRanFinalization;
        if (shouldRunGC) {
            justRanFinalization = false;
        } else {
            runGC = true;
        }
    }
    if (shouldRunGC) {
        Runtime.getRuntime().gc();
    }
}


public static void runFinalization() {
  boolean shouldRunGC;
  synchronized (LOCK) {
    shouldRunGC = runGC;
    runGC = false;
  }
  if (shouldRunGC) {
    Runtime.getRuntime().gc();
  }
  Runtime.getRuntime().runFinalization();
  synchronized (LOCK) {
    justRanFinalization = true;
  }
}
```



### 整体代码实现

```kotlin
class LeakCanaryApplication : Application() {

    /**
     * 整体思路：我们利用java 弱引用对象在GC回收的时，如果该对象仅存在弱引用关系，必定会被垃圾回收器回收
     * 而弱引用对象可以配合引用队列一起使用，如果对象只存在弱引用关系的话，被GC回收时，会将该对象移动到引用队列中
     * 所以，我们可以定一个Map(key为className＋hashCode，value为该对象的弱引用)，用来存放所有的需要观察内存泄漏的对象，在每次
     * 往Map添加弱引用观察对象时，都将该对象与弱引用队列关联起来。如果在GC回收后，我们能从弱引用队列查到有回收对象，将该对象
     * 从Map中删除，最后Map中剩下的元素就是发生了内存泄漏的对象
     */

    val TAG = "leakCanary测试"

    /**
     * step 1: 定义一个弱引用的Queue，用于配合弱引用对象监测该对象内存回首情况
     */
    val referenceQueue : ReferenceQueue<Any> = ReferenceQueue()
    /**
     * step 2: 定义一个Map，用于观察对象的回收情况，key是对象的className＋hashCode，value就是该对象的弱引用，HashKeyWeakReference是继承的WeakReference
     */
    private val watchedMap = mutableMapOf<String, HashKeyWeakReference>()



    override fun onCreate() {
        super.onCreate()

        registerActivityLifecycleCallbacks(object :ActivityLifecycleCallbacks{
            override fun onActivityPaused(activity: Activity?) {
                Log.d(TAG,"onActivityPaused")
            }

            override fun onActivityResumed(activity: Activity?) {
                Log.d(TAG,"onActivityResumed")
            }

            override fun onActivityStarted(activity: Activity?) {
                Log.d(TAG,"onActivityStarted")
            }

            override fun onActivityDestroyed(activity: Activity?) {
                if(activity!=null){
                    /**
                     * step 3: 在Activity即将销毁时，我们将该Activity使用弱引用对象观察并加入弱引用队列
                     */
                    val key = activity.javaClass.simpleName+activity.hashCode()
                    val hashKeyWeakReference = HashKeyWeakReference(activity,key,referenceQueue)
                    watchedMap[key] = hashKeyWeakReference
                    Log.d(TAG,"activity销毁->开启内存泄漏监测->将该Activity加入弱引用观察队列->watchMap key = $key")
                    /**
                     * step 4: 开始调用系统GC回收垃圾
                     */
                    startGCAndCheckLeak(2000)
                }

            }

            override fun onActivitySaveInstanceState(activity: Activity?, outState: Bundle?) {
                Log.d(TAG,"onActivitySaveInstanceState")
            }

            override fun onActivityStopped(activity: Activity?) {
                Log.d(TAG,"onActivityStopped")
            }

            override fun onActivityCreated(activity: Activity?, savedInstanceState: Bundle?) {
                Log.d(TAG,"onActivityCreated")
            }

        })
    }

     fun startGCAndCheckLeak(time : Long){
        Handler(Looper.getMainLooper()).postDelayed({
            //调用系统GC，回收垃圾，其中就包括我们在 onActivityDestroyed时创建的  WeakReference(activity)对象
            //经过GC后，我们去referenceQueue里面看看有没有被回收的对象，有的话，则说明这个弱引用对象是正常回收了
            Runtime.getRuntime().gc()
            removeWeaklyReachableObjects()
            /**
             * step 6: 最后观察内存泄漏对象的Map里剩下的元素，就是发生了内存泄漏的对象
             */
            watchedMap.forEach {
                Log.d(TAG,"GC回收垃圾后->发现内存泄漏对象->watchMap key = ${it.key}")
                watchedMap.remove(it.key)
            }
        },time)
    }

    private fun removeWeaklyReachableObjects() {

        /**
         * step 5: GC之后，从referenceQueue中不断获取元素，该元素就代表正常销毁了的弱引用对象，我们将该对象从观察的Map中移除掉
         */
        var ref: HashKeyWeakReference?
        do {
            ref = referenceQueue.poll() as HashKeyWeakReference?
            if (ref != null) {
                watchedMap.remove(ref.key)
            }
        } while (ref != null)
    }
}
```