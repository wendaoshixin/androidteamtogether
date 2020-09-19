## **前言**

为了更好的执行开发规范，提升审查效率，专业线全体小伙伴都将采用自动化脚本对代码进行扫描。

考虑时间和扫描成本，经过专业线集体商议，采用`每次提交(P3C扫描)+每个迭代(Lint扫描)`的方案。

- **每次代码提交**：每位小伙伴,代码git提交前，必须使用P3C进行静态扫描。若存在问题，修改好后，才能进行提交。

- **每次版本迭代优化**：项目的Android负责人，必须对项目进行一次Lint检查，将存在问题分配给组内小伙伴，进行修复。

## **1. 阿里P3C插件**

> p3c是阿里巴巴提供的Java代码检测规范，并且提供了Android Studio插件供我们使用，除了可以进行扫描检查之外，还有实时代码规范提示功能。

#### **1.1插件安装**

通过Jetbrains官方仓库安装

1. 打开 Settings >> Plugins >> Browse repositories...

![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/p3c_1.png)

2. 在搜索框输入alibaba即可看到Alibaba Java Code Guidelines插件，点击Install进行安装，然后重启IDE生效

![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/p3c_2.png)

---

#### **1.2插件使用**

目前插件实现了开发手册中的的53条规则，大部分基于PMD实现，其中有4条规则基于IDEA实现，并且基于IDEA [Inspection](https://www.jetbrains.com/help/idea/code-inspection.html)实现了实时检测功能。部分规则实现了Quick Fix功能，对于可以提供Quick Fix但没有提供的，我们会尽快实现，也欢迎有兴趣的同学加入进来一起努力。 目前插件检测有两种模式：实时检测、手动触发。

##### 实时检测

实时检测功能会在开发过程中对当前文件进行检测，并以高亮的形式提示出来，同时也可以支持Quick Fix，该功能默认开启，可以通过配置关闭。

##### 结果高亮提示

检测结果高亮提示，并且鼠标放上去会弹出提示信息。

![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/p3c_3.png)

![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/p3c_4.png)

##### 智能QuickFix功能

Alt+Enter键可呼出Intention菜单，不同的规则会提示不同信息的Quick Fix按钮

![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/p3c_5.png)

##### 关闭实时检测

在某些情况下，我们不希望对代码提示违规信息，比如我们在阅读Github开源项目代码的时候，如果界面出现一堆红色、黄色的提示，此时心里肯定是飘过一万只草泥马。这个时候我们可以通过Inspection的设置关闭实时检测功能。

1. 通过右键快速关闭（打开）所有规则的实时检测功能

 ![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/p3c_6.png) 

2. 通过Settings >> Editor >> Inspections 进行手动设置

![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/p3c_7.png)

也可以关闭某条规则的实时检测功能或者修改提示级别。

#### **1.3代码扫描**

可以通过右键菜单、Toolbar按钮、快捷键三种方式手动触发代码检测。同时结果面板中可以对部分实现了QuickFix功能的规则进行快速修复。

##### 触发扫描

在当前编辑的文件中点击右键，可以在弹出的菜单中触发对该文件的检测。

![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/p3c_8.png)

在左侧的Project目录树种点击右键，可以触发对整个工程或者选择的某个目录、文件进行检测。

![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/p3c_9.png)

##### 扫描结果

检测结果直接使用IDEA Run Inspection By Name功能的结果界面，插件的检测结果分级为Blocker、Critical、Major。默认按等级分组，方便统计每个级别错误的数量。

![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/p3c_10.png)

默认情况我们在结果面板需要双击具体违规项才能打开对应的源文件，开启Autoscroll To Source选项，单击面板中的文件名、或者是具体的违规项的时候IDEA会自动打开对应的源文件。

![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/p3c_11.png)

##### QuickFix

对于实现Quick Fix的规则，在结果面板中可以直接一键修复 `注意：IDEA14、15可以通过左下角的灯泡进行一键修复操作。`

![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/p3c_12.png)

![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/p3c_13.png)

#### **1.4代码提交时检测**

1. 在提交代码框勾选Alibaba Code Guidelines选项 

![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/p3c_14.png) 

2. 如果有违反手册的地方会提示是否继续提交，选择取消后会自动对修改的代码进行扫描 

![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/p3c_15.png) 


## **2. Android Lint**

### **2.1检索**

快捷键：Analyze —— Inspect Code

* whole project:整个项目

* modul 'xxxx':某个模块

* file 'xxxx':某个文件

* custom scope:自定义范围

![lint_1.jpeg](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/lint_1.jpeg)

检查完毕后会弹出 Inspection 的控制台，并在其中列出详细的检查结果：

![lint_2.jpeg](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/lint_2.jpeg)

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

![lint_3.jpeg](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/lint_3.jpeg)

如上图所示，系统会提示所导致的错误规则，包括未配置的国际化语言

### **2.2配置**

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

![line_3.webp](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/line_3.webp)

issue 标签中使用 **id 指定一个规则**，severity="ignore" 则表明禁用这个规则。需要注意的是，某些规则可以通过 ignore 标签指定仅对某些属性禁用，例如上面的 Deprecated，表示检查是否有使用不推荐的属性和方法，而在 issue 标签中包裹一个 ignore 标签，在 ignore 标签的 regexp 属性中使用正则表达式指定了 singleLine，则表明对 singleLine 这个属性屏蔽检查。

另外开发者也可以使用 @SuppressLint(issue id) 标注针对某些代码忽略某些 Lint 检查，这个标注既可以加到成员变量之前，也可以加到方法声明和类声明之前，分别针对不同范围进行屏蔽。



## **3. PMD代码静态检查（可选）**

因P3C是基于PMD基础上开发的java规范插件，因此不考虑自定义PMD的扫描规则，若是感兴趣，可以自行研究。

### **3.1安装**：

1. 自动安装：file --> settings --> plugins  搜索  pmd


![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E5%BB%BA%E8%AE%BE%E6%8F%90%E8%AE%AE/picture/pmd_1.png)

2. 下载进行安装：

[PMDPlugin - Plugins | JetBrains](https://plugins.jetbrains.com/plugin/1137-pmdplugin)

### **3.2IDE 插件直接使用**：

1. pmd支持自定义规则集，配置规则：


![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/pmd_2.png)

2. 用户可以选择在单个或者多个文件或文件夹上运行PMD：


-  选中 文件或文件夹 --> 右击 --> Run PDM --> 选择规则集
  
  ![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/pmd_3.png)
  
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

  ![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/pmd_4.png)

4. 开始pmd检查，并显示检查的结果。


![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/pmd_5.png)

### **3.3 gradle插件方式使用**：

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

  ![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/pmd_6.png)

3. 执行检测，点击pmd任务或者执行命令行执行

  ```undefined
  ./gradlew check
  ```

  ![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/pmd_7.png)

4. 查看检查结果
   
    ![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/pmd_8.png)
    
5. 结果为html或者xml格式，自行配置
   
    ![](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/落地方案/文档/picture/pmd_9.png)