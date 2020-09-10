###### Android Lint

##### 检索

快捷键：Analyze —— Inspect Code

* whole project:整个项目

* modul 'xxxx':某个模块

* file 'xxxx':某个文件

* custom scope:自定义范围

![lint_1.jpeg](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E5%BB%BA%E8%AE%BE%E6%8F%90%E8%AE%AE/picture/lint_1.jpeg)

检查完毕后会弹出 Inspection 的控制台，并在其中列出详细的检查结果：

![lint_2.jpeg](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E5%BB%BA%E8%AE%BE%E6%8F%90%E8%AE%AE/picture/lint_2.jpeg)

如上图所展示的，Android Lint 对检查的结果进行了分类，同一个规则（issue）下的问题会聚合，其中针对 Android 的规则类别会在分类前说明是 Android 相关的，主要是六类：

- **Accessibility 无障碍**，例如 ImageView 缺少contentDescription 描述，String 编码字符串等问题。
- **Correctness 正确性**
- **Internationalization 国际化**，如字符缺少翻译等问题。
- **Performance 性能**，例如在 onMeasure、onDraw 中执行 new，内存泄露，产生了冗余的资源，xml 结构冗余等。
- **Security 安全性**，例如没有使用 HTTPS 连接 Gradle，AndroidManifest 中的权限问题等。
- **Usability 易用性**，例如缺少某些倍数的切图，重复图标等。  

其他的结果条目则是**针对 Java 语法**的问题，另外每一个问题都有区分严重程度（severity），从高到底依次是：  

Fatal  

Error  

Warning  

Information  

Ignore  

其中 Fatal 和 Error 都是指错误，但是 Fatal 类型的错误会直接中断 ADT 导出 APK，更为严重。  

在结果列表中点击一个条目，可以看到详细的源文件名和位置，以及命中的错误规则（issue）、解决方案或者屏蔽提示

![lint_3.jpeg](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E5%BB%BA%E8%AE%BE%E6%8F%90%E8%AE%AE/picture/lint_3.jpeg)

如上图所示，系统会提示所导致的错误规则，包括未配置的国际化语言

##### 配置

对于执行 Lint 操作的相关配置，是定义在 gradle 文件的 lintOptions 中，可定义的选项及其默认值

```
android {
    lintOptions {
        // 设置为 true，则当 Lint 发现错误时停止 Gradle 构建
        abortOnError false
        // 设置为 true，则当有错误时会显示文件的全路径或绝对路径 (默认情况下为true)
        absolutePaths true
        // 仅检查指定的问题（根据 id 指定）
        check 'NewApi', 'InlinedApi'
        // 设置为 true 则检查所有的问题，包括默认不检查问题
        checkAllWarnings true
        // 设置为 true 后，release 构建都会以 Fatal 的设置来运行 Lint。
        // 如果构建时发现了致命（Fatal）的问题，会中止构建（具体由 abortOnError 控制）
        checkReleaseBuilds true
        // 不检查指定的问题（根据问题 id 指定）
        disable 'TypographyFractions','TypographyQuotes'
        // 检查指定的问题（根据 id 指定）
        enable 'RtlHardcoded','RtlCompat', 'RtlEnabled'
        // 在报告中是否返回对应的 Lint 说明
        explainIssues true
        // 写入报告的路径，默认为构建目录下的 lint-results.html
        htmlOutput file("lint-report.html")
        // 设置为 true 则会生成一个 HTML 格式的报告
        htmlReport true
        // 设置为 true 则只报告错误
        ignoreWarnings true
        // 重新指定 Lint 规则配置文件
        lintConfig file("default-lint.xml")
        // 设置为 true 则错误报告中不包括源代码的行号
        noLines true
        // 设置为 true 时 Lint 将不报告分析的进度
        quiet true
        // 覆盖 Lint 规则的严重程度，例如：
        severityOverrides ["MissingTranslation": LintOptions.SEVERITY_WARNING]
        // 设置为 true 则显示一个问题所在的所有地方，而不会截短列表
        showAll true
        // 配置写入输出结果的位置，格式可以是文件或 stdout
        textOutput 'stdout'
        // 设置为 true，则生成纯文本报告（默认为 false）
        textReport false
        // 设置为 true，则会把所有警告视为错误处理
        warningsAsErrors true
        // 写入检查报告的文件（不指定默认为 lint-results.xml）
        xmlOutput file("lint-report.xml")
        // 设置为 true 则会生成一个 XML 报告
        xmlReport false
        // 将指定问题（根据 id 指定）的严重级别（severity）设置为 Fatal
        fatal 'NewApi', 'InlineApi'
        // 将指定问题（根据 id 指定）的严重级别（severity）设置为 Error
        error 'Wakelock', 'TextViewEdits'
        // 将指定问题（根据 id 指定）的严重级别（severity）设置为 Warning
        warning 'ResourceAsColor'
        // 将指定问题（根据 id 指定）的严重级别（severity）设置为 ignore
        ignore 'TypographyQuotes'
    }
}
```

**lint.xml** 这个文件则是配置 Lint 需要禁用哪些规则（issue），以及自定义规则的严重程度（severity），lint.xml 文件是通过 **issue 标签**指定对一个规则的控制，在**项目根目录**中建立一个 lint.xml 文件后 **Android Lint 会自动识别该文件**，在执行检查时按照 lint.xml 的内容进行检查。如上面提到的那样，开发者也可以通过 lintOptions 中的 lintConfig 选项来指定配置文件。一个 lint.xml 示例如下：

![line_3.webp](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E5%BB%BA%E8%AE%BE%E6%8F%90%E8%AE%AE/picture/line_3.webp)

issue 标签中使用 **id 指定一个规则**，severity="ignore" 则表明禁用这个规则。需要注意的是，某些规则可以通过 ignore 标签指定仅对某些属性禁用，例如上面的 Deprecated，表示检查是否有使用不推荐的属性和方法，而在 issue 标签中包裹一个 ignore 标签，在 ignore 标签的 regexp 属性中使用正则表达式指定了 singleLine，则表明对 singleLine 这个属性屏蔽检查。

另外开发者也可以使用 @SuppressLint(issue id) 标注针对某些代码忽略某些 Lint 检查，这个标注既可以加到成员变量之前，也可以加到方法声明和类声明之前，分别针对不同范围进行屏蔽。


**PMD代码静态检查**

##### 安装：

1. 自动安装：file --> settings --> plugins  搜索  pmd
  

![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E5%BB%BA%E8%AE%BE%E6%8F%90%E8%AE%AE/picture/pmd_1.png)

2. 下载进行安装：
  

[PMDPlugin - Plugins | JetBrains](https://plugins.jetbrains.com/plugin/1137-pmdplugin)

##### IDE 插件直接使用：

1. pmd支持自定义规则集，配置规则：
  

![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E5%BB%BA%E8%AE%BE%E6%8F%90%E8%AE%AE/picture/pmd_2.png)

2. 用户可以选择在单个或者多个文件或文件夹上运行PMD：
  

-  选中 文件或文件夹 --> 右击 --> Run PDM --> 选择规则集
  
  ![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E5%BB%BA%E8%AE%BE%E6%8F%90%E8%AE%AE/picture/pmd_3.png)
  
- 常用的规则定义
  
  ```
  <div>
  <?xml version="1.0"?>
  <ruleset xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" name="alibaba-pmd"
      xmlns="http://pmd.sourceforge.net/ruleset/2.0.0"
      xsi:schemaLocation="http://pmd.sourceforge.net/ruleset/2.0.0 http://pmd.sourceforge.net/ruleset_2_0_0.xsd">
      <rule ref="rulesets/java/ali-concurrent.xml"/>
      <rule ref="rulesets/java/ali-comment.xml"/>
      <rule ref="rulesets/java/ali-naming.xml">
      </rule>
      <rule ref="rulesets/java/ali-constant.xml">
      </rule>
      <rule ref="rulesets/java/ali-other.xml"/>
      <rule ref="rulesets/java/ali-orm.xml"/>
      <rule ref="rulesets/java/ali-exception.xml"/>
      <rule ref="rulesets/java/ali-oop.xml">
      </rule>
      <rule ref="rulesets/java/ali-set.xml"/>
      <rule ref="rulesets/java/ali-flowcontrol.xml">
      </rule>
  </ruleset>
  </div>
  ```
  

3. 另外一种执行方式：Tools -> PMD -> PreDefined
  
  ![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E5%BB%BA%E8%AE%BE%E6%8F%90%E8%AE%AE/picture/pmd_4.png)
  
4. 开始pmd检查，并显示检查的结果。
  

![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E5%BB%BA%E8%AE%BE%E6%8F%90%E8%AE%AE/picture/pmd_5.png)

##### gradle插件方式使用：

1. gradle插件
  
  ```
  apply plugin: 'pmd'
  
  
  check.dependsOn 'pmd'
  
  def configDir = "${project.rootDir}/scripts"
  def reportsDir = "${project.buildDir}/reports"
  
  print("reportsDir=${reportsDir}")
  
  task pmd(type: Pmd) {
      ignoreFailures = true
      ruleSetFiles = files("$configDir/pmd/pmd-ruleset.xml")
      ruleSets = []
  
      source 'src'
      include '**/*.java'
      exclude '**/gen/**'
  
      reports {
          xml.enabled = false
          html.enabled = true
          xml {
              destination file("$reportsDir/pmd/pmd.xml")
          }
          html {
              destination file("$reportsDir/pmd/pmd.html")
          }
      }
  }
  ```
  
2. 模块引用插件
  
  ![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E5%BB%BA%E8%AE%BE%E6%8F%90%E8%AE%AE/picture/pmd_6.png)
  
3. 执行检测，点击pmd任务或者执行命令行执行
  
  ```undefined
  ./gradlew check
  ```
  
  ![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E5%BB%BA%E8%AE%BE%E6%8F%90%E8%AE%AE/picture/pmd_7.png)
  
4. 查看检查结果
    
    ![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E5%BB%BA%E8%AE%BE%E6%8F%90%E8%AE%AE/picture/pmd_8.png)
    
5. 结果为html或者xml格式，自行配置
      
    ![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E5%BB%BA%E8%AE%BE%E6%8F%90%E8%AE%AE/picture/pmd_9.png)