

**前言**：

- 按照分模块进行分类记录问题
- 按照产生原因、解决方式来记录问题。

## **UI相关问题**







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

