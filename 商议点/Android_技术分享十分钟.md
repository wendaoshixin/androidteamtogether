# **Android 技术分享十分钟模版**

[TOC]

#### **一、主题**

先简述主题：

> 渲染API(如OpenGL, OpenGL ES, OpenVG)和本地窗口系统之间的接口。它处理图形上下文管理，表面/缓冲区创建，绑定和渲染同步，并使用其他Khronos API实现高性能，加速，混合模式2D和3D渲染OpenGL / OpenGL ES渲染客户端API OpenVG渲染客户端API原生平台窗口系统。

分享主题：技术、行业资讯

#### **二、使用场景：**

- 场景1: 自定义相机。

- 场景2:  自定义播放器。

- 场景3:  x x x x x x

#### **三、方案的优劣(可选)：**

若是有需要，可以列举正反案例或者优势，进行对比。

正例：

反例：

或者列举优势：

- 优势1
- 优势2

#### **四、功能介绍：**

绘图的基本步骤：

![直播渲染技术—EGL解析- 图玩智能科技的个人空间- OSCHINA](https://oscimg.oschina.net/oscnet/up-93cf4ca49f349c2deed9e984e916e4342ff.png)

实现步骤(大纲)：

- 绘制模块
- 初始化显示设备
- 配置选项

- 接下来。。。。。。

接下来，根据每个步骤逐一进行介绍。

1.首先我们需要知道绘制内容的目标在哪里，EGLDisplayer是一个封装系统屏幕的数据类型，通常通过eglGetDisplay方法来返回EGLDisplay作为OpenGl ES的渲染目标，eglGetDisplay()

```cpp
 if ( (mEGLDisplay = EGL14.eglGetDisplay(EGL14.EGL_DEFAULT_DISPLAY)) == EGL14.EGL_NO_DISPLAY) {
                throw new RuntimeException("unable to get EGL14 display");
            }
```



2.初始化显示设备，第一参数代表Major版本，第二个代表Minor版本。如果不关心版本号，传0或者null就可以了。初始化与 EGLDisplay 之间的连接：eglInitialize()

```cpp
            if (!EGL14.eglInitialize(mEGLDisplay, 0, 0)) {
                throw new RuntimeException("unable to initialize EGL14");
            }
```



3.下面我们进行配置选项，使用eglChooseConfig()方法，Android平台的配置代码如下：

```cpp
int[] attribList = {
                    EGL14.EGL_RED_SIZE, 8,
                    EGL14.EGL_GREEN_SIZE, 8,
                    EGL14.EGL_BLUE_SIZE, 8,
                    EGL14.EGL_ALPHA_SIZE, 8,
                    EGL14.EGL_RENDERABLE_TYPE, EGL14.EGL_OPENGL_ES2_BIT,
                    EGL_RECORDABLE_ANDROID, 1,
                    EGL14.EGL_NONE
            };
            EGLConfig[] configs = new EGLConfig[1];
            int[] numConfigs = new int[1];
            EGL14.eglChooseConfig(mEGLDisplay, attribList, 0, configs, 0, configs.length,
                    numConfigs, 0);
```

#### **五、源码分析（可选）**

适当添加一些源码或者分析的插图。

若是源码篇幅过长，可作为一个系列，进行分次进行分享。

#### **六、提问互动环节（可选）**

- 列举一些疑问，与专业线小伙伴进行互动。
  - 疑问1: 为什么会
  - 疑问2: 为什么不会

- 让专业线小伙伴来提问。

#### **七、总结(可选)**

- 个人总结：使用该技术，提升了开发效率。
- 个人观点：这是未来技术的趋势



