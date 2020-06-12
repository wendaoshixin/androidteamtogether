## **App 黑白化的实现思路**

4 月 4 日这一天，不少 网站、App 都实现了黑白化。

![Image text](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E6%8A%80%E6%9C%AF%E5%88%86%E4%BA%AB%E4%BC%9A%E8%AE%AE/picture/%E9%BB%91%E7%99%BD%E5%9B%BE.png)


那app开发黑白化是不是比较麻烦呢？大家普遍的思路是：

 - 换肤
 - 展示 server 下发的图片，还要单独做灰度处理

这么看起来工作量还是很大的。


#### **那我们如何给app页面加一个灰度效果呢？**

构思过程
#####。
#####。
#####。

我们的 app 页面正常情况下，其实也是 Canvas 绘制出来的对吧？

Canvas 对应的相关 API 肯定也是支持灰度的。

那么是不是我们在控件绘制的时候，比如 draw 之前设置个灰度效果就可以呢？

基于这个思路我们来验证一下：

#### **给ImageView上个灰度效果**

首先我们来编写个自定义的 ImageView ,取名: GrayImageView

布局文件 与 原始对比图：
![Image text](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E6%8A%80%E6%9C%AF%E5%88%86%E4%BA%AB%E4%BC%9A%E8%AE%AE/picture/%E5%B8%83%E5%B1%80%E6%96%87%E4%BB%B6.png)

GrayImageView 的代码 与 编译对比图：

![对比图](/Users/lm9/Desktop/对比图.png)