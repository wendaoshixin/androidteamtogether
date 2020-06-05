# 规约

- 为了减少他人阅读你代码的痛苦值，请在关键地方做好注释。
- 注释必须简洁，描述逻辑清晰，杜绝长篇大论
- 注释量不少于代码总量的三分之一
- 除变量定义等较短语句的注释可用行尾注释外，其他注释应当避免使用行尾注释



# 注释规范

1. #### 类注释【强制】

   每个类完成后应该有作者姓名，联系方式，创建时间，描述，版本号，版权所有的注释

   ```java
   /**
    * desc   : xxxx 描述
    * author : Blankj
    * e-mail : xxx@xx
    * date   : 2020/06/03
    * version: 1.0
    * 版权所有:雷漫网络科技
    */
   public class WelcomeActivity {
       ...
   }
   ```
   

具体可以在 AS 中自己配制，进入 Settings -> Editor -> File and Code Templates -> Includes -> File Header，输入以下文本即可

```java
   /**
    * desc   :
    * author : ${USER}
    * e-mail : xxx@xx
    * date   : ${YEAR}/${MONTH}/${DAY}
    * version: 1.0
    * 版权所有:雷漫网络科技
    */
```

   

2. #### 方法注释【强制】

   每一个成员方法（包括自定义成员方法、覆盖方法、属性方法）的方法头都必须做方法头注释，在方法前一行输入 `/** + 回车` 或者设置 `Fix doc comment`（Settings -> Keymap -> Fix doc comment）快捷键，AS 便会帮你生成模板，我们只需要补全参数即可，如下所示。

   ```java
   /**
    * 方法的一句话概述
    * <p>方法详述（简单方法可不必详述）</p>
    * @param s 说明参数含义
    * @return 说明返回值含义
    * @throws IOException 说明发生此异常的条件
    * @throws NullPointerException 说明发生此异常的条件
    */
   ```

   例：

   ```java
   /**
    * bitmap 转 byteArr
    *
    * @param bitmap bitmap 对象
    * @param format 格式
    * @return 字节数组
    */
   public static byte[] bitmap2Bytes(@NotNull Bitmap bitmap, @NotNull CompressFormat format) {
       if (bitmap == null) return null;
       ByteArrayOutputStream baos = new ByteArrayOutputStream();
       bitmap.compress(format, 100, baos);
       return baos.toByteArray();
   }
   ```

   方法注释中不允许出现@params, @return的参数描述错误的情况, 必须实时更新。【强制】

   一段逻辑建议使用//的方式【推荐】

   方法/参数建议添加 @Nullable, @NotNull, @UiThread 等注解【推荐】

   

3. #### 块注释【强制】

   块注释与其周围的代码在同一缩进级别。它们可以是 `/* ... */` 风格，也可以是 `// ...` 风格（**`//` 后最好带一个空格**）。对于多行的 `/* ... */` 注释，后续行必须从 `*` 开始， 并且与前一行的 `*` 对齐。以下示例注释都是 OK 的。

   ```java
   /*
    * This is
    * okay.
    */
   
   // And so
   // is this.
   
   /* 
    * Or you can
    * even do this. */
   ```

   变量注释不允许使用与类/方法一致的注释形式。

   在写多行注释时，如果你希望在必要时能重新换行（即注释像段落风格一样），那么使用 `/* ... */`。

   

4. #### 其他注释【参考】

   a. AS 已帮你集成了一些注释模板，我们只需要直接使用即可，在代码中输入 `todo`、`fixme` 等这些注释模板，回车后便会出现如下注释。

   ```java
   // TODO: 17/3/14 需要实现，但目前还未实现的功能的说明
   // FIXME: 17/3/14 需要修正，甚至代码是错误的，不能工作，需要修复的说明
   ```

   b. 如果当前layout 或资源需要被多处调用，或公共使用的layout（如：list_item），则需要在xml写明注释

   c. 引用的第三方库尽量注释其作用，github或博客地址。
