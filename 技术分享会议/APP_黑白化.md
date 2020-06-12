## **App 黑白化的实现思路**

4 月 4 日这一天，不少 网站、App 都实现了黑白化。

![Image text](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E6%8A%80%E6%9C%AF%E5%88%86%E4%BA%AB%E4%BC%9A%E8%AE%AE/picture/%E9%BB%91%E7%99%BD%E5%9B%BE.png)


那app开发黑白化是不是比较麻烦呢？大家普遍的思路是：

 - 换肤
 - 展示 server 下发的图片，还要单独做灰度处理

这么看起来工作量还是很大的。


#### **那我们如何给app页面加一个灰度效果呢？**

构思过程
##### **。**
##### **。**
##### **。**

我们的 app 页面正常情况下，其实也是 Canvas 绘制出来的对吧？

Canvas 对应的相关 API 肯定也是支持灰度的。

那么是不是我们在控件绘制的时候，比如 draw 之前设置个灰度效果就可以呢？

基于这个思路我们来验证一下：

#### **给ImageView上个灰度效果**

首先我们来编写个自定义的 ImageView ,取名: GrayImageView

布局文件 与 原始对比图：
![Image text](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E6%8A%80%E6%9C%AF%E5%88%86%E4%BA%AB%E4%BC%9A%E8%AE%AE/picture/%E5%B8%83%E5%B1%80%E6%96%87%E4%BB%B6.png)

GrayImageView 的代码 与 编译对比图：
![Image text](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E6%8A%80%E6%9C%AF%E5%88%86%E4%BA%AB%E4%BC%9A%E8%AE%AE/picture/%E5%AF%B9%E6%AF%94%E5%9B%BE.png)

从效果图看说明上面的实现思路是正确的。

我们再看一下上面的代码，代码非常简单。

我们复写了draw 方法，在该方法中给canvas 做了一下特殊处理。

什么特殊处理呢？其实就是设置了一个灰度效果

```Java
setSaturation();
```

为何这行代码能起到这样的效果呢？看一下源码

```Java
 /**
     * Set the matrix to affect the saturation of colors.
     *
     * @param sat A value of 0 maps the color to gray-scale. 1 is identity.
     */
public void setSaturation(float sat) {
        reset();
        float[] m = mArray;

        final float invSat = 1 - sat;
        final float R = 0.213f * invSat;
        final float G = 0.715f * invSat;
        final float B = 0.072f * invSat;

        m[0] = R + sat; m[1] = G;       m[2] = B;
        m[5] = R;       m[6] = G + sat; m[7] = B;
        m[10] = R;      m[11] = G;      m[12] = B + sat;
    }
```

在 App中，我们对于颜色的处理很多时候会采用颜色矩阵，是一个4*5的矩阵，应用到一个具体的颜色[R, G, B, A]上。

我们来看一下 Android 提供给我们的 API ColorMartrix类。[API链接](https://developer.android.com/reference/android/graphics/ColorMatrix)

```Java
/**
 * 4x5 matrix for transforming the color and alpha components of a Bitmap.
 * The matrix can be passed as single array, and is treated as follows:
 *
 * <pre>
 *  [ a, b, c, d, e,
 *    f, g, h, i, j,
 *    k, l, m, n, o,
 *    p, q, r, s, t ]</pre>
 *
 * <p>
 * When applied to a color <code>[R, G, B, A]</code>, the resulting color
 * is computed as:
 * </p>
 *
 * <pre>
 *   R&rsquo; = a*R + b*G + c*B + d*A + e;
 *   G&rsquo; = f*R + g*G + h*B + i*A + j;
 *   B&rsquo; = k*R + l*G + m*B + n*A + o;
 *   A&rsquo; = p*R + q*G + r*B + s*A + t;</pre>
 *
 * <p>
 * That resulting color <code>[R&rsquo;, G&rsquo;, B&rsquo;, A&rsquo;]</code>
 * then has each channel clamped to the <code>0</code> to <code>255</code>
 * range.
 * </p>
 *
 * <p>
 * The sample ColorMatrix below inverts incoming colors by scaling each
 * channel by <code>-1</code>, and then shifting the result up by
 * <code>255</code> to remain in the standard color space.
 * </p>
 *
 * <pre>
 *   [ -1, 0, 0, 0, 255,
 *     0, -1, 0, 0, 255,
 *     0, 0, -1, 0, 255,
 *     0, 0, 0, 1, 0 ]</pre>
 */
    }
```

所以想要达到我们的效果就通过调用饱和度API就能够实现。

今天就讲到这里

##### **。**
##### **。**
##### **。**

你们可以尝试一下 TextView、Button 的效果，原理也是一样的。

或者通过 BaseActivity 做一下整个app 的黑白效果也是可以。

```Java
@Override
    public View onCreateView(String name, Context context, AttributeSet attrs) {
        if ("FrameLayout".equals(name)) {
            int count = attrs.getAttributeCount();
            for (int i = 0; i < count; i++) {
                String attributeName = attrs.getAttributeName(i);
                String attributeValue = attrs.getAttributeValue(i);
                if (attributeName.equals("id")) {
                    int id = Integer.parseInt(attributeValue.substring(1));
                    String idVal = getResources().getResourceName(id);
                    if ("android:id/content".equals(idVal)) {
                        //因为这是API23之后才能改变的，所以你的判断版本
                        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
                            //获取窗口区域
                            Window window = this.getWindow();
                            window.addFlags(WindowManager.LayoutParams.FLAG_DRAWS_SYSTEM_BAR_BACKGROUNDS);
                            //设置状态栏颜色
                              window.setStatusBarColor(Color.parseColor("#848484"));
                        }
                        GrayFrameLayout grayFrameLayout = new GrayFrameLayout(context, attrs);
                        return grayFrameLayout;
                    }
                }
            }
        }
        return super.onCreateView(name, context, attrs);

}
```






