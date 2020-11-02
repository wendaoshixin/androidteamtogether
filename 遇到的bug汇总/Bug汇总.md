**前言**：

- 按照分模块进行分类记录问题
- 按照产生原因、解决方式来记录问题。

## **UI相关问题**
### **1.RecyclerView notifyItemInserted(0)在RecycleView数据未满一屏的时候，会有插入动画效果，但当RecycleView数据超过一屏时，就不再有Insert的动画效果了。**

**产生原因**：
>执行notifyItemInserted(0)方法后，旧的第位置0变为了位置1（在屏幕上可见），而新添加的位置0则变为不可见，需要手动向下滑动才可见。

**解决方式**：
```插入item后，将列表滚动到顶部位置
   adapter.addData(0);
   adapter.notifyItemInserted(0);
   recyclerView.scrollToPosition(0);
```


## **内存泄漏与内存优化相关问题**





## **冷启动相关问题**

### **1.黑白屏WindowBackgroud优化与今日头条适配方案冲突**

**产生原因**：

>黑白屏适配是通过设置Theme中`<item name="android:windowBackground">@drawable/x x x x</item>方式来处理，即通过带主题背景的启动页来处理。详情点击，[谷歌官方启动优化方案](https://developer.android.com/topic/performance/vitals/launch-time)。
>
>今日头条的屏幕适配方案是在Actvitiy的onCreate()执行后，会对屏幕density进行修改，从而导致会导致控件重绘，造成闪烁。

**解决方式**：

定义一个注解，用于判断过滤。

```java
    /**
     *  用于标记某个Activity不适配
     */
    @Target(ElementType.TYPE)
    @Retention(RetentionPolicy.RUNTIME)
    public @interface  Ignore{

    }
    private static boolean isIntercept(Activity activity){
        return activity.getClass().isAnnotationPresent(Ignore.class);
    }
```

## **ANR相关问题**

### **1.Android 8.0启动前台服务产生ANR**

**产生原因**：
>因为在Application里面启动了一个音乐播放器的服务，调用context.startForegroundService(intent)，但是这个地方会直接卡住，产生ANR
>
>

```kotlin
        mConnection = MusicPlayerServiceConnection()
        mConnection?.let {
            val intent = Intent(context, MusicPlayerService::class.java)
            context.bindService(
                intent, it,
                Context.BIND_AUTO_CREATE
            )
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                //启动前台服务
                context.startForegroundService(intent)
            } else {
                context.startService(intent)
            }

            callBack.onSuccess()
        }
```
**解决方式**：

8.0 以后不希望后台应用运行后台服务，一旦通过startForegroundService() 启动前台服务，必须在service 中有startForeground() 配套，不然会出现ANR 或者crash
可以在startForeground() 配置notification 即可

9 对话框或item布局中使用ConstraintLayout后，布局经常显示不出来，可换成其他布局Framelayout等等。








