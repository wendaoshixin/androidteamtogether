# Android UI规范

## 一、全局规范

1.【强制】文字样式，项目开发之初就要抽取设计稿中标题文字等的通用样式，预先定义通用样式。这些样式主要包括：文字大小，颜色，背景，边距等，视情况而定。  

2.【推荐】图片样式，UI切图的时候要贴边切，而开发中使用这些图片的时候，就要定义几个规范样式。同样需要定义图片的大小、圆角的等样式。  

3.【推荐】控件的使用，优先使用系统View、组合view、自定义view。  

4.【推荐】布局原则：减少层级，推荐使用`<include/>、<merge/>、<ViewStub/>、ConstraintLayout`。

5.【推荐】UI更新，子线程尽量不要更改UI界面，如果出现程序运行时UI不能更新，崩溃且没有明显的语法错误，可以检查是否在非UI创建线程更新了UI。

6.【推荐】SurfaceView或TextureView可以通过将绘图操作移动到另一个单独线程上提高性能。如果某些绘图操作影响性能，这时可以考虑使用SurfaceView和TextureView，因为SurfaceView在常规视图系统之外，所以无法像常规试图一样移动、缩放或旋转一个SurfaceView。TextureView是Android4.0引入的，除了与SurfaceView一样在单独线程绘制外，还可以像常规视图一样被改变。

## 二、注意细节

1.【推荐】布局中不得不使用 ViewGroup 多重嵌套时，不要使用 LinearLayout 嵌套， 改用 ConstraintLayout/RelativeLayout，可以有效降低嵌套数。 
说明： 
Android 应用页面上任何一个 View 都需要经过 measure、layout、draw 三个步骤 才能被正确的渲染。从 xml layout 的顶部节点开始进行 measure，每个子节点都需 要向自己的父节点提供自己的尺寸来决定展示的位置，在此过程中可能还会重新 measure（由此可能导致 measure 的时间消耗为原来的 2-3 倍）。节点所处位置越 深，套嵌带来的 measure 越多，计算就会越费时。这就是为什么扁平的 View 结构 会性能更好。 
同时，页面拥上的 View越多，measure、layout、draw所花费的时间就越久。要缩 短这个时间，关键是保持 View的树形结构尽量扁平，而且要移除所有不需要渲染的 View。理想情况下，总共的 measure，layout，draw时间应该被很好的控制在 16ms 以内，以保证滑动屏幕时 UI 的流畅。 
要找到那些多余的 View（增加渲染延迟的 view），可以用 Layout Inspector/Hierarachy Viewer 工具，可视化的查看所有的 view。 
正例： 

```
<?xml version="1.0" encoding="utf-8"?> 
<android.support.constraint.ConstraintLayout> 
    <RelativeLayout> 
    <TextView/> 
    ... 
    <ImageView/> 
    </RelativeLayout> 
</android.support.constraint.ConstraintLayout>
```
 
反例： 

```
<LinearLayout> 
    <LinearLayout> 
        <RelativeLayout> 
        <TextView/> 
        ... 
        <ImageView/> 
        </RelativeLayout> 
    </LinearLayout> 
</LinearLayout>
```
多重嵌套导致 measure 以及 layout 等步骤耗时过多。 
扩展参考： 
1) https://developer.android.com/studio/profile/hierarchy-viewer.html 
2) http://mrpeak.cn/android/2016/01/11/android-performance-ui 
3) https://www.safaribooksonline.com/library/view/high-performance-android/97 81491913994/ch04.html#figure-story_tree 

2.【推荐】在 Activity 中显示对话框或弹出浮层时，尽量使用 DialogFragment，而非 Dialog/AlertDialog，这样便于随Activity生命周期管理对话框/弹出浮层的生命周期。 
正例： 

```
public void showPromptDialog(String text){ 
    DialogFragment promptDialog = new DialogFragment() { 
        @Override 
        public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) { 
            getDialog().requestWindowFeature(Window.FEATURE_NO_TITLE); 
            View view = inflater.inflate(R.layout.fragment_prompt, container); 
            return view; 
        } 
    };
  promptDialog.show(getFragmentManager(), text); 
}
```

3.【推荐】源文件统一采用 UTF-8 的形式进行编码。  

4.【推荐】文本大小使用单位 sp，view 大小使用单位 dp。对于 Textview，如果在文 字大小确定的情况下推荐使用 wrap_content 布局避免出现文字显示不全的适配问题。    

5.【推荐】禁止在设计布局时多次设置子 view 和父 view 中为同样的背景造成页面过 度绘制，推荐将不需要显示的布局进行及时隐藏。 
正例： 

```
<?xml version="1.0" encoding="utf-8"?> 
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android" 
    android:layout_width="fill_parent" 
    android:layout_height="fill_parent" 
    android:orientation="vertical" > 
<TextView 
    android:layout_width="fill_parent" 
    android:layout_height="wrap_content" 
    android:text="@string/hello" /> 
<Button  
    android:layout_width="fill_parent" 
    android:layout_height="wrap_content" 
    android:text="click it !" 
    android:id="@+id/btn_mybuttom" /> 
<ImageView  
    android:id="@+id/img" 
    android:layout_width="fill_parent" 
    android:layout_height="wrap_content"
      android:visibility="gone" 
    android:src="@drawable/youtube" /> 
<TextView  
    android:text="it is an example!" 
    android:layout_width="fill_parent" 
    android:layout_height="wrap_content" /> 
</LinearLayout> 
```
反例： 

```
@Override 
protected void onDraw(Canvas canvas) { 
    super.onDraw(canvas); 
    int width = getWidth(); 
    int height = getHeight(); 
    mPaint.setColor(Color.GRAY); 
    canvas.drawRect(0, 0, width, height, mPaint); 
    mPaint.setColor(Color.CYAN); 
    canvas.drawRect(0, height/4, width, height, mPaint); 
    mPaint.setColor(Color.DKGRAY); 
    canvas.drawRect(0, height/3, width, height, mPaint); 
    mPaint.setColor(Color.LTGRAY); 
    canvas.drawRect(0, height/2, width, height, mPaint); 
}
```

6.【推荐】在需要时刻刷新某一区域的组件时，建议通过以下方式避免引发全局 layout 刷新: 
1) 设置固定的 view大小的高宽，如倒计时组件等； 
2) 调用 view的 layout 方式修改位置，如弹幕组件等； 
3) 通过修改 canvas 位置并且调用 invalidate(int l, int t, int r, int b)等方式限定刷新 区域； 
4) 通过设置一个是否允许 requestLayout 的变量，然后重写控件的 requestlayout、 onSizeChanged 方法，判断控件的大小没有改变的情况下，当进入 requestLayout 的时候，直接返回而不调用 super 的 requestLayout 方法。 

7.【推荐】不能在 Activity没有完全显示时显示 PopupWindow 和 Dialog。 

8.【推荐】尽量不要使用 AnimationDrawable，它在初始化的时候就将所有图片加载 到内存中，特别占内存，并且还不能释放，释放之后下次进入再次加载时会报错。 
说明： 
Android 的帧动画可以使用 AnimationDrawable 实现，但是如果你的帧动画中如果 包含过多帧图片，一次性加载所有帧图片所导致的内存消耗会使低端机发生 OOM 异常。帧动画所使用的图片要注意降低内存消耗，当图片比较大时，容易出现 OOM。 
正例： 
图片数量较少的 AnimationDrawable 还是可以接受的。 

```
<?xml version="1.0" encoding="utf-8"?> 
<animation-list xmlns:android="http://schemas.android.com/apk/res/android" android:oneshot ="true"> 
    <item android:duration="500" android:drawable="@drawable/ic_heart_100"/> 
    <item android:duration="500" android:drawable="@drawable/ic_heart_75"/> 
    <item android:duration="500" android:drawable="@drawable/ic_heart_50"/> 
    <item android:duration="500" android:drawable="@drawable/ic_heart_25"/> 
    <item android:duration="500" android:drawable="@drawable/ic_heart_0"/> 
</animation-list>
```
反例： 
```
<animation-list xmlns:android="http://schemas.android.com/apk/res/android" android:oneshot ="false">
<item android:drawable="@drawable/soundwave_new_1_40" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_41" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_42" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_43" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_44" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_45" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_46" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_47" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_48" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_49" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_50" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_51" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_52" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_53" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_54" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_55" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_56" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_57" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_58" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_59" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_60" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_61" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_62" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_63" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_64" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_65" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_66" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_67" android:duration="100" />
     <item android:drawable="@drawable/soundwave_new_1_68" android:duration="100" /> 
    <item android:drawable="@drawable/soundwave_new_1_69" android:duration="100" /> 
</animation-list> 
```
上述如此多图片的动画就不建议使用 AnimationDrawable 了。 
扩展参考： 
1) https://stackoverflow.com/questions/8692328/causing-outofmemoryerror-in-fr ame-by-frame-animation-in-android 
2) http://blog.csdn.net/wanmeilang123/article/details/53929484 
3) https://segmentfault.com/a/1190000005987659 
4) https://developer.android.com/reference/android/graphics/drawable/Animatio nDrawable.html

9.【强制】不能使用 ScrollView 包裹 ListView/GridView/ExpandableListVIew;因为这 样会把 ListView 的所有 Item 都加载到内存中，要消耗巨大的内存和 cpu 去绘制图 面。 
说明： 
ScrollView中嵌套 List 或 RecyclerView的做法官方明确禁止。除了开发过程中遇到 的各种视觉和交互问题，这种做法对性能也有较大损耗。ListView 等 UI 组件自身有 垂直滚动功能，也没有必要在嵌套一层 ScrollView。目前为了较好的 UI 体验，更贴 近 Material Design 的设计，推荐使用 NestedScrollView。 
正例：

```
<?xml version="1.0" encoding="utf-8"?> 
<LinearLayout> 
    <android.support.v4.widget.NestedScrollView> 
        <LinearLayout> 
            <ImageView/> 
            ... 
            <android.support.v7.widget.RecyclerView/> 
        </LinearLayout> 
    </android.support.v4.widget.NestedScrollView>
</LinearLayout>     
```
反例： 

```
<ScrollView> 
    <LinearLayout> 
        <TextView/> 
        ... 
        <ListView/> 
        <TextView /> 
    </LinearLayout> 
</ScrollView>
```
扩展参考： 
1) https://developer.android.com/reference/android/widget/ScrollView.html 
2) https://developer.android.com/reference/android/support/v4/widget/NestedSc rollView.html