# Android 资源文件命名与使用

**1.layout 文件的命名方式【推荐】**

  
*   Activity 的 layout 以 activity_ 开头

  
*   Fragment 的 layout 以 fragment_ 开头

  
*   Dialog 的 layout 以 dialog_ 开头

  
*   include 的 layout 以 include_ 开头

  
*   ListView 的行 layout 以 list_item_ 开头

   
*  RecyclerView 的 item layout 以 recycle_item_ 开头

   
*  GridView 的 item layout 以 grid_item_ 开头

   

**2.drawable 资源命名方式【推荐】**

   
*  小写单词+下划线的方式命名，如下所示。

   ```java
   selector_game_detail_collection.xml
   ```

  
*   drawable中一般存放selector,shape等内容

   

**3.mipmap命名方式【推荐】**

   
*  mipmap中只存放图片内容，以小写单词+下划线+图片状态 的方式命名，规则如下

   

   ```java
   模块名_业务功能描述_控件描述_控件状态词
   ```

   
*  示例

   ```java
    bg_charge_amount_default.png
    bg_charge_amount_selected.png
    bg_charge_amount_presseded.png
   ```

   
*  Tip：根据分辨率不同存放 在不同的 mipmap 目录下，例如 mipmap-xxhdpi。

   

**4.anim 资源名称以小写单词+下划线的方式命名【推荐】**

   
*  anim中存放动画资源，采用规则如下。

   ```java
   模块名_业务功能_[方向|序号]
   ```

   
*  Tip：tween 动画资源:尽可能以通用的动画名称命名，如 dialog_fade_in ,dialog_fade_out (动画+方向);
        frame 动画资源:尽可能以模 块+功能命名+序号。如:module_loading_grey_001

   

**5.color 资源使用#AARRGGBB 格式，写入 colors.xml 文件中【推荐】**

   
*  规则:

       ```java
       模块名_逻辑名称_颜色
       ```

  
*   如下所示：
    
       ```java
       <color name="dialog_progress_bg_color">#5F000000</color>
       ```

   

**6.dimen 资源以小写单词+下划线方式命名，写入dimens.xml 文件中【推荐】**

   
*  规则：

   ```java
   模块名_描述信息
   ```

   
*  如下所示：

   ```java
   <dimen name="splash_logo_width">120dp</dimen>
   ```

   

**7**.style 资源采用小写首字母大写的方式命名，写入styles.xml 文件中【推荐】

   
*  规则：

   ```java
   "描述+模块"
   ```

   
*  如下所示：

   ```java
   <style name="TransparentActivity" parent="AppTheme">
           <item name="android:windowBackground">@android:color/transparent</item>
           <item name="android:windowIsTranslucent">true</item>
           <item name="android:windowNoTitle">true</item>
           <item name="android:windowAnimationStyle">@style/TransparentAnimation</item>
   </style>
   ```

   

**8.String 资源文件或者文本用到字符需要全部写入strings.xml 文件,用下划线分割【推荐】**

   
*  规则：

   ```java
   模块名_逻辑名称
   ```

   
*  如下所示：

   ```java
   <string name="game_setting_show_mouse">显示鼠标</string>
   ```

**9.id命名， 使用小写单词+下划线分割【推荐】**

   
*  规则：

   ```java
   控件缩写_模块名
   ```

  
*  如下所示：

   ```java
   android:id="@+id/tv_game_time"
   ```

   

  
*   Tip 常用控件缩写方式：

   

   |       控件       | 缩写 |
   | :--------------: | ---- |
   |   LinearLayout   | ll   |
   |  RelativeLayout  | rl   |
   | ConstraintLayout | cl   |
   |     ListView     | lv   |
   |    ScollView     | sv   |
   |     TextView     | tv   |
   |      Button      | btn  |
   |    ImageView     | iv   |
   |     CheckBox     | cb   |

   

**10.图片存放资源文件规则【强制】**

    |  HVGA（480*320 ）  |  mdpi   |
    | :----------------: | :-----: |
    |  WVGA（800*480）   |  hdpi   |
    | FWVGA（854*480*）  |  hdpi   |
    |   QHD（960*540）   |  hdpi   |
    |  720P（1280*720）  |  xhdpi  |
    | 1080P（1920*1080） | xxhdpi  |
    |     更高分辨率     | xxxhdpi |


*  Tip: 根据当前的设备屏幕尺寸和密度，将会寻找最匹配的资源，如果将高分辨率图片放
        入低密度目录，将会造成低端机加载过大图片资源，又可能造成 OOM，同时也是资
        源浪费，没有必要在低端机使用大图。

     