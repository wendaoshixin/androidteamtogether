Android 技术分享十分钟

**一、主题：EGL**

渲染API(如OpenGL, OpenGL ES, OpenVG)和本地窗口系统之间的接口。它处理图形上下文管理，表面/缓冲区创建，绑定和渲染同步，并使用其他Khronos API实现高性能，加速，混合模式2D和3D渲染OpenGL / OpenGL ES渲染客户端API OpenVG渲染客户端API原生平台窗口系统。

**二、作用：**

1. 与设备的原生窗口系统通信。
2. 查询绘图表面的可用类型和配置。
3. 创建绘图表面。
4. 在OpenGL ES 和其他图形渲染API之间同步渲染。
5. 管理纹理贴图等渲染资源。

使用场景：自定义相机，自定义播放器

**三、功能介绍：**

绘图的基本步骤：

![直播渲染技术—EGL解析- 图玩智能科技的个人空间- OSCHINA](https://oscimg.oschina.net/oscnet/up-93cf4ca49f349c2deed9e984e916e4342ff.png)

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



4.接下来我们需要创建OpenGl的上下文环境 EGLContext 实例，这里值得留意的是，OpenGl的任何一条指令都是必须在自己的OpenGl上下文环境中运行，我们可以通过eglCreateContext()方法来构建上下文环境：

```cpp
    int[] attrib_list = {
                    EGL14.EGL_CONTEXT_CLIENT_VERSION, 2,
                    EGL14.EGL_NONE
            };
            mEGLContext = EGL14.eglCreateContext(mEGLDisplay, configs[0], EGL14.EGL_NO_CONTEXT,
                    attrib_list, 0);
```

eglCreateContext中的第三个参数可以传入一个EGLContext类型的变量，改变量的意义是可以与正在创建的上下文环境共享OpenGl资源，包括纹理ID,FrameBuffer以及其他Buffer资源。如果没有的话可以填写Null.



5.通过上面四步,获取OpenGl 上下文之后，说明EGL和OpenGl ES端的环境已经搭建完毕，也就是说OpengGl的输出我们可以获取到了。下面的步骤我们讲如何将EGl和设备屏幕连接起来。如果连接呢？当然，这时候我们就要使用EGLSurface了,我们通过EGL库提供eglCreateWindowSurface可以创建一个实际可以显示的surface.当然，如果需要离线的surface，我们可以通过eglCreatePbufferSurface创建。eglCreateWindowSurface()

```cpp
     private EGLSurface mEGLSurface = EGL14.EGL_NO_SURFACE;
      int[] surfaceAttribs = {
                    EGL14.EGL_NONE
            };
            mEGLSurface = EGL14.eglCreateWindowSurface(mEGLDisplay, configs[0], mSurface,
                    surfaceAttribs, 0);
```



6.通过上面的步骤，EGL的准备工作做好了，一方面我们为OpenGl ES渲染提供了目标及上下文环境，可以接收到OpenGl ES渲染出来的纹理，另一方面我们连接好了设备显示屏（这里指SurfaceView或者TextureView）,接下来我们讲解如何在创建好的EGL环境下工作的。首先我们有一点必须要明确，OpenGl ES 的渲染必须新开一个线程，并为该线程绑定显示设备及上下文环境（Context）。因为前面有说过OpenGl指令必须要在其上下文环境中才能执行。所以我们首先要通过 eglMakeCurrent()方法来绑定该线程的显示设备及上下文。

```css
EGL14.eglMakeCurrent(mEGLDisplay, mEGLSurface, mEGLSurface, mEGLContext);
```



7.当我们绑定完成之后，我们就可以进行RenderLoop循环了。这里简单说一下，EGL的工作模式是双缓冲模式，其内部有两个FrameBuffer(帧缓冲区，可以理解为一个图像存储区域)，当EGL将一个FrameBuffer显示到屏幕上的时候，另一个FrameBuffer就在后台等待OpenGl ES进行渲染输出。知道调用了eglSwapBuffers这条指令的时候，才会把前台的FrameBuffers和后台的FrameBuffer进行交换，这样界面呈现的就是OpenGl ES刚刚渲染的结构了。

```css
mInputSurface.swapBuffers();
```



8.当然，在所有的操作都执行完之后，我们要销毁资源。特别注意，销毁资源必须在当前线程中进行，不然会报错滴。首先我们销毁显示设备（EGLSurface）,然后销毁上下文（EGLContext）,停止并释放线程，最后终止与EGLDisplay之间的链接，

```css
 EGL14.eglDestroySurface(mEGLDisplay, mEGLSurface);
                EGL14.eglDestroyContext(mEGLDisplay, mEGLContext);
                EGL14.eglReleaseThread();
                EGL14.eglTerminate(mEGLDisplay);
```



四、源码分析（主要为关联性强的事件）

GLSurfaceView关联、绘制过程的源码

![image-20200817171321393](/Users/lm45/Library/Application Support/typora-user-images/image-20200817171321393.png)

![image-20200817170541907](/Users/lm45/Library/Application Support/typora-user-images/image-20200817170541907.png)

备注：如果时间允许的情况下，可以开发一个demo出来，作为感兴趣的童鞋后期可以一起研讨。

