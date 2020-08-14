## 自定义 EGL



## 目录

- EGL与OpenGL的关系

- 系统的OpenGL

- 自定义EGL

- OpenGL 渲染YUV



#### EGL与OpenGL的关系

![Image text](http://192.168.11.214:8087/android-team/androidteamtogether/raw/master/%E6%8A%80%E6%9C%AF%E5%88%86%E4%BA%AB%E4%BC%9A%E8%AE%AE/picture/EGL%E4%B8%8EOpenGL%E5%85%B3%E7%B3%BB%E5%9B%BE.png)

注：上面的关系不仅限于Android平台，IOS、Windows等其他平台也是一样的。



#### 系统的OpenGL

```
GLSurfaceView.java
```

```java
GLSurfaceView gl = new GLSurfaceView(this);
GLSurfaceView.Renderer renderer = new GLSurfaceView.Renderer() {
    @Override
    public void onSurfaceCreated(GL10 gl, EGLConfig config) {
    }

    @Override
    public void onSurfaceChanged(GL10 gl, int width, int height) {
    }

    @Override
    public void onDrawFrame(GL10 gl) {
    }
};
gl.setRenderer(renderer);
```

```java
public void setRenderer(Renderer renderer) {
    checkRenderThreadState();
    if (mEGLConfigChooser == null) {
        mEGLConfigChooser = new SimpleEGLConfigChooser(true);
    }
    if (mEGLContextFactory == null) {
        mEGLContextFactory = new DefaultContextFactory();
    }
    if (mEGLWindowSurfaceFactory == null) {
        mEGLWindowSurfaceFactory = new DefaultWindowSurfaceFactory();
    }
    mRenderer = renderer;
    mGLThread = new GLThread(mThisWeakRef);
    mGLThread.start();
}
```

```java
@Override
public void run() {
    setName("GLThread " + getId());
    if (LOG_THREADS) {
        Log.i("GLThread", "starting tid=" + getId());
    }

    try {
        guardedRun();
    } catch (InterruptedException e) {
        // fall thru and exit normally
    } finally {
        sGLThreadManager.threadExiting(this);
    }
}
```

```java
private void guardedRun() throws InterruptedException {
    mEglHelper = new EglHelper(mGLSurfaceViewWeakRef);
    mHaveEglContext = false;
    mHaveEglSurface = false;
    mWantRenderNotification = false;

    try {
     ...
     
      while (true) {
          synchronized (sGLThreadManager) {
              while (true) {
                  if (mShouldExit) {
                      return;
                  }
                  ...
                            
                   // If we don't have an EGL context, try to acquire one.
                   if (! mHaveEglContext) {
                       if (askedToReleaseEglContext) {
                          askedToReleaseEglContext = false;
                       } else {
                           try {
                              mEglHelper.start();
                           } catch (RuntimeException t) {
                              sGLThreadManager.releaseEglContextLocked(this);
                              throw t;
                           }
                           mHaveEglContext = true;
                           createEglContext = true;

                        	 sGLThreadManager.notifyAll();
												}
										}
                    ...
                 } // while
              }   //  synchronized 
              
              if (createEglSurface) {
                  if (LOG_SURFACE) {
                  			Log.w("GLThread", "egl createSurface");
                  }
                  if (mEglHelper.createSurface()) {
                    	synchronized(sGLThreadManager) {
                          mFinishedCreatingEglSurface = true;
                          sGLThreadManager.notifyAll();
                  		}
                  } else {
                      synchronized(sGLThreadManager) {
                      mFinishedCreatingEglSurface = true;
                      mSurfaceIsBad = true;
                      sGLThreadManager.notifyAll();
                  }
                  continue;
                  }
                  createEglSurface = false;
              }
              
              int swapError = mEglHelper.swap();
              switch (swapError) {
                  case EGL10.EGL_SUCCESS:
                  break;
                  case EGL11.EGL_CONTEXT_LOST:
                  default:
              }
              ...
         } // while
         ...
    }
    ...
}
```

```java
/**
 * Initialize EGL for a given configuration spec.
 * @param configSpec
 */
public void start() {
    if (LOG_EGL) {
        Log.w("EglHelper", "start() tid=" + Thread.currentThread().getId());
    }
    /*
     * Get an EGL instance
     */
    mEgl = (EGL10) EGLContext.getEGL();

    /*
     * Get to the default display.
     */
    mEglDisplay = mEgl.eglGetDisplay(EGL10.EGL_DEFAULT_DISPLAY);

    if (mEglDisplay == EGL10.EGL_NO_DISPLAY) {
        throw new RuntimeException("eglGetDisplay failed");
    }

    /*
     * We can now initialize EGL for that display
     */
    int[] version = new int[2];
    if(!mEgl.eglInitialize(mEglDisplay, version)) {
        throw new RuntimeException("eglInitialize failed");
    }
    GLSurfaceView view = mGLSurfaceViewWeakRef.get();
    if (view == null) {
        mEglConfig = null;
        mEglContext = null;
    } else {
        mEglConfig = view.mEGLConfigChooser.chooseConfig(mEgl, mEglDisplay);

        /*
        * Create an EGL context. We want to do this as rarely as we can, because an
        * EGL context is a somewhat heavy object.
        */
        mEglContext = view.mEGLContextFactory.createContext(mEgl, mEglDisplay, mEglConfig);
    }
    if (mEglContext == null || mEglContext == EGL10.EGL_NO_CONTEXT) {
        mEglContext = null;
        throwEglException("createContext");
    }
    if (LOG_EGL) {
        Log.w("EglHelper", "createContext " + mEglContext + " tid=" + 
              Thread.currentThread().getId());
    }

    mEglSurface = null;
}

/**
 * Initialize EGL for a given configuration spec.
 * @param configSpec
 */
  public void start() {
    if (LOG_EGL) {
      Log.w("EglHelper", "start() tid=" + Thread.currentThread().getId());
    }
    /*
               * Get an EGL instance
               */
    mEgl = (EGL10) EGLContext.getEGL();

    /*
               * Get to the default display.
               */
    mEglDisplay = mEgl.eglGetDisplay(EGL10.EGL_DEFAULT_DISPLAY);

    if (mEglDisplay == EGL10.EGL_NO_DISPLAY) {
      throw new RuntimeException("eglGetDisplay failed");
    }

   /*
    * We can now initialize EGL for that display
    */
   int[] version = new int[2];
   if(!mEgl.eglInitialize(mEglDisplay, version)) {
      throw new RuntimeException("eglInitialize failed");
   }
   GLSurfaceView view = mGLSurfaceViewWeakRef.get();
   if (view == null) {
    	mEglConfig = null;
    	mEglContext = null;
   } else {
      mEglConfig = view.mEGLConfigChooser.chooseConfig(mEgl, mEglDisplay);
      /*
       * Create an EGL context. We want to do this as rarely as we can, because an
       * EGL context is a somewhat heavy object.
       */
     mEglContext = view.mEGLContextFactory.createContext(mEgl, mEglDisplay,
                                                        mEglConfig);
  }
  if (mEglContext == null || mEglContext == EGL10.EGL_NO_CONTEXT) {
    mEglContext = null;
    throwEglException("createContext");
  }
  if (LOG_EGL) {
    Log.w("EglHelper", "createContext " + mEglContext + " tid=" +
          Thread.currentThread().getId());
  }

  mEglSurface = null;
}
```

```java
/**
 * Display the current render surface.
 * @return the EGL error code from eglSwapBuffers.
 */
public int swap() {
    if (! mEgl.eglSwapBuffers(mEglDisplay, mEglSurface)) {
        return mEgl.eglGetError();
    }
    return EGL10.EGL_SUCCESS;
}
```

总结：OpenGL 是平台相关的，同时，也是线程相关的。



#### 自定义EGL

1、得到默认的显示设备 -- eglGetDisplay

2、初始化默认显示设备 -- eglInitialize

3、设置显示设备的属性

4、从系统中获取对应属性的配置 -- eglChooseConfig

5、创建EglContext -- eglCreateContext

6、创建渲染的Surface -- eglCreateWindowSurface

7、绑定EglContext和Surface到显示设备中 -- eglMakeCurrent

8、刷新数据，显示渲染场景 -- eglSwapBuffers

###### 源代码：

LmEglHelper.h

```c++
#include "EGL/egl.h"
#include "../util/LmLog.h"

class LmEglHelper {
public:
    EGLDisplay mEglDisplay;
    EGLSurface mEglSurface;
    EGLConfig  mEglConfig;
    EGLContext mEglContext;

public:
    LmEglHelper();
    ~LmEglHelper();

    int initEgl(EGLNativeWindowType win); // 初始化 EGL

    int swapBuffers(); // 数据交换

    void destoryEgl(); // 回收销毁

};
```

LmEglHelper.cpp

```c++
#include "LmEglHelper.h"

LmEglHelper::LmEglHelper() {
    mEglDisplay = EGL_NO_DISPLAY;
    mEglSurface = EGL_NO_SURFACE;
    mEglContext = EGL_NO_CONTEXT;
    mEglConfig = NULL;
}

LmEglHelper::~LmEglHelper() {

}

int LmEglHelper::initEgl(EGLNativeWindowType window) {
    //1、得到默认的显示设备
    mEglDisplay = eglGetDisplay(EGL_DEFAULT_DISPLAY);
    if (mEglDisplay == EGL_NO_DISPLAY) {
        LOGE("eglGetDisplay error");
        return -1;
    }

    //2、初始化默认显示设备
    EGLint *version = new EGLint[2];
    if (!eglInitialize(mEglDisplay, &version[0], &version[1])) {
        LOGE("eglInitialize error");
        return -1;
    }

    //3、设置显示设备的属性
    const EGLint attribs[] = {
            EGL_RED_SIZE, 8,
            EGL_GREEN_SIZE, 8,
            EGL_BLUE_SIZE, 8,
            EGL_ALPHA_SIZE, 8,
            EGL_DEPTH_SIZE, 8,
            EGL_STENCIL_SIZE, 8,
            EGL_RENDERABLE_TYPE, EGL_OPENGL_ES2_BIT,
            EGL_NONE
    };

    EGLint num_config;
    if (!eglChooseConfig(mEglDisplay, attribs, NULL, 1, &num_config)) {
        LOGE("eglChooseConfig  error 1");
        return -1;
    }

    //4、从系统中获取对应属性的配置
    if (!eglChooseConfig(mEglDisplay, attribs, &mEglConfig, num_config, &num_config)) {
        LOGE("eglChooseConfig  error 2");
        return -1;
    }

    int attrib_list[] = {
            EGL_CONTEXT_CLIENT_VERSION, 2,
            EGL_NONE
    };
	  //5、创建EglContext
    mEglContext = eglCreateContext(mEglDisplay, mEglConfig, EGL_NO_CONTEXT, attrib_list);
    if (mEglContext == EGL_NO_CONTEXT) {
        LOGE("eglCreateContext  error");
        return -1;
    }

    //6、创建渲染的Surface
    mEglSurface = eglCreateWindowSurface(mEglDisplay, mEglConfig, window, NULL);
    if (mEglSurface == EGL_NO_SURFACE) {
        LOGE("eglCreateWindowSurface  error");
        return -1;
    }

    //7、绑定EglContext和Surface到显示设备中
    if (!eglMakeCurrent(mEglDisplay, mEglSurface, mEglSurface, mEglContext)) {
        LOGE("eglMakeCurrent  error");
        return -1;
    }

    LOGD("egl init success! ");
    return 0;
}
//8、刷新数据，显示渲染场景
int LmEglHelper::swapBuffers() {
    if (mEglDisplay != EGL_NO_DISPLAY && mEglSurface != EGL_NO_SURFACE) {
        if (eglSwapBuffers(mEglDisplay, mEglSurface)) {
            return 0;
        }
    }

    return -1;
}

void LmEglHelper::destoryEgl() {
    if (mEglDisplay != EGL_NO_DISPLAY) {
        eglMakeCurrent(mEglDisplay, EGL_NO_SURFACE, EGL_NO_SURFACE, EGL_NO_CONTEXT);
    }

    if (mEglDisplay != EGL_NO_DISPLAY && mEglSurface != EGL_NO_SURFACE) {
        eglDestroySurface(mEglDisplay, mEglSurface);
        mEglSurface = EGL_NO_SURFACE;
    }

    if (mEglDisplay != EGL_NO_DISPLAY && mEglContext != EGL_NO_CONTEXT) {
        eglDestroyContext(mEglDisplay, mEglContext);
        mEglContext = EGL_NO_CONTEXT;
    }

    if (mEglDisplay != EGL_NO_DISPLAY) {
        eglTerminate(mEglDisplay);
        mEglDisplay = EGL_NO_DISPLAY;
    }
}
```

LmEglThread.h

```c++
#include <EGL/eglplatform.h>
#include "pthread.h"
#include "android/native_window.h"
#include "LmEglHelper.h"
#include <unistd.h>
#include "GLES2/gl2.h"

#define OPENGL_RENDER_AUTO 1
#define OPENGL_RENDER_HANDLE 2

class LmEglThread {
public:
    pthread_t eglThread = -1;
    ANativeWindow *nativeWindow = NULL;

    bool isCreate = false;
    bool isChange = false;
    bool isExit = false;
    bool isStart = false;

    bool surfaceWidth = 0;
    bool surfaceHeight = 0;

    typedef void(*OnCreate)(void *);
    OnCreate onCreate;
    void *onCreteCtx;

    typedef void(*OnChange)(int width, int height, void *);
    OnChange onChange;
    void *onChangeCtx;

    typedef void(*OnDraw)(void *);
    OnDraw onDraw;
    void *onDrawCtx;

    typedef void(*OnDestroy)(void *);
    OnDestroy onDestroy;
    void *onDestroyctx;

    int renderType = OPENGL_RENDER_AUTO;

    pthread_mutex_t pthread_mutex;
    pthread_cond_t pthread_cond;

public:
    LmEglThread();
    ~LmEglThread();

    void onSurfaceCreate(EGLNativeWindowType window);

    void onSurfaceChange(int width, int height);

    void callBackOnCreate(OnCreate onCreate, void *ctx);

    void callBackOnChange(OnChange onChange, void *ctx);

    void callBackOnDraw(OnDraw onDraw, void *ctx);

    void setRenderType(int renderType);

    void notifyRender();

    void destroy();

};
```

LmEglThread.cpp

```c++
LmEglThread::LmEglThread() {
    pthread_mutex_init(&pthread_mutex, NULL);
    pthread_cond_init(&pthread_cond, NULL);
}

LmEglThread::~LmEglThread() {
    pthread_mutex_destroy(&pthread_mutex);
    pthread_cond_destroy(&pthread_cond);
}

void *eglThreadImpl(void *context) {
    LmEglThread *lmEglThread = static_cast<LmEglThread *>(context);

    if (lmEglThread != NULL) {
        LmEglHelper *eglHelper = new LmEglHelper();
        eglHelper->initEgl(lmEglThread->nativeWindow);
        lmEglThread->isExit = false;
        while (true) {
            if (lmEglThread->isCreate) {
                LOGD("eglthread call surfaceCreate");
                lmEglThread->isCreate = false;
                lmEglThread->onCreate(lmEglThread->onCreteCtx);
            }

            if (lmEglThread->isChange) {
                LOGD("eglthread call surfaceChange");
                lmEglThread->isChange = false;
                lmEglThread->onChange(lmEglThread->surfaceWidth,
                                      lmEglThread->surfaceHeight,
                                      lmEglThread->onChangeCtx);
                lmEglThread->isStart = true;
            }

            //
            LOGD("draw");
            if (lmEglThread->isStart) {
                lmEglThread->onDraw(lmEglThread->onDrawCtx);
                eglHelper->swapBuffers();
            }
            if (lmEglThread->renderType == OPENGL_RENDER_AUTO) {
                usleep(1000000 / 60);
            } else {
                pthread_mutex_lock(&lmEglThread->pthread_mutex);
                pthread_cond_wait(&lmEglThread->pthread_cond, &lmEglThread->pthread_mutex);
                pthread_mutex_unlock(&lmEglThread->pthread_mutex);
            }

            if (lmEglThread->isExit) {
                lmEglThread->onDestroy(lmEglThread->onDestroyctx);

                eglHelper->destoryEgl();
                delete eglHelper;
                break;
            }
        }
    }

    return 0;
}

void LmEglThread::onSurfaceCreate(EGLNativeWindowType window) {
    if (eglThread == -1) {
        isCreate = true;
        nativeWindow = window;

        pthread_create(&eglThread, NULL, eglThreadImpl, this);
    }
}

void LmEglThread::onSurfaceChange(int width, int height) {
    isChange = true;
    surfaceWidth = width;
    surfaceHeight = height;

    notifyRender();
}

void LmEglThread::callBackOnCreate(LmEglThread::OnCreate onCreate, void *ctx) {
    this->onCreate = onCreate;
    this->onCreteCtx = ctx;
}

void LmEglThread::callBackOnChange(LmEglThread::OnChange onChange, void *ctx) {
    this->onChange = onChange;
    this->onChangeCtx = ctx;
}

void LmEglThread::callBackOnDraw(LmEglThread::OnDraw onDraw, void *ctx) {
    this->onDraw = onDraw;
    this->onDrawCtx = ctx;
}

void LmEglThread::setRenderType(int renderType) {
    this->renderType = renderType;
}

void LmEglThread::notifyRender() {
    pthread_mutex_lock(&pthread_mutex);
    pthread_cond_signal(&pthread_cond);
    pthread_mutex_unlock(&pthread_mutex);
}

void LmEglThread::destroy() {
    isExit = true;
    notifyRender();
    pthread_join(eglThread, NULL);
    nativeWindow = NULL;
    eglThread = -1;
}
```

意义：获取mEglContext。mEglContext可以支持纹理共享。



#### 渲染YUV

LmBaseOpengl.h

```c++
#include <GLES2/gl2.h>
#include <cstring>
#include "../util/LmLog.h"

class LmBaseOpengl {

public:

    int surface_width;
    int surface_height;

    char * vertex;
    char * fragment;

    float *vertexs;
    float *fragments;

    GLuint program;
    GLuint vShader;
    GLuint fShader;

public:
    LmBaseOpengl();
    ~LmBaseOpengl();

    virtual void onCreate();

    virtual void onChange(int w, int h);

    virtual void draw();

    virtual void destroy();

    virtual void destorySorce();

    virtual void setPilex(void *data, int width, int height, int length);

    virtual void setYuvData(void *y, void *u, void *v, int width, int height);
};
```

LmBaseOpengl.cpp

```c++
#include "LmBaseOpengl.h"

LmBaseOpengl::LmBaseOpengl() {
    vertexs = new float[8];
    fragments = new float[8];

    float v[] = {1,-1,
                 1,1,
                 -1,-1,
                 -1,1};
    memcpy(vertexs, v, sizeof(v));

    float f[] = {1,1,
                 1,0,
                 0,1,
                 0,0};
    memcpy(fragments, f, sizeof(f));
}

LmBaseOpengl::~LmBaseOpengl() {

    delete []vertexs;
    delete []fragments;
}

void LmBaseOpengl::onCreate() {
}

void LmBaseOpengl::onChange(int w, int h) {
}

void LmBaseOpengl::draw() {
}

void LmBaseOpengl::destroy() {
}

void LmBaseOpengl::setPilex(void *data, int width, int height, int length) {
}

void LmBaseOpengl::destorySorce() {
}

void LmBaseOpengl::setYuvData(void *y, void *u, void *v, int width, int height) {
}
```

LmOpengl.h

```c++
#include "../egl/LmEglThread.h"
#include "android/native_window.h"
#include "android/native_window_jni.h"
#include "LmBaseOpengl.h"
#include "LmPlayYUV.h"

class LmOpengl {

public:
    LmEglThread *eglThread = NULL;
    ANativeWindow *nativeWindow = NULL;
    LmBaseOpengl *baseOpengl = NULL;

    void *pilex = NULL;

public:
    LmOpengl();
    ~LmOpengl();

    void onCreateSurface(JNIEnv *env, jobject surface);

    void onChangeSurface(int width, int height);

    void onDestorySurface();

    void setYuvData(void *y, void *u, void *v, int w, int h);
};
```

LmOpengl.cpp

```c++
#include "LmOpengl.h"

void callback_SurfaceCrete(void *ctx) {

    LmOpengl *wlOpengl = static_cast<LmOpengl *>(ctx);

    if (wlOpengl != NULL) {
        if (wlOpengl->baseOpengl != NULL) {
            wlOpengl->baseOpengl->onCreate();
        }
    }
}

void callback_SurfacChange(int width, int height, void *ctx) {
    LmOpengl *wlOpengl = static_cast<LmOpengl *>(ctx);
    if (wlOpengl != NULL) {
        if (wlOpengl->baseOpengl != NULL) {
            wlOpengl->baseOpengl->onChange(width, height);
        }
    }
}

void callback_SurfaceDraw(void *ctx) {
    LmOpengl *wlOpengl = static_cast<LmOpengl *>(ctx);
    if (wlOpengl != NULL) {
        if (wlOpengl->baseOpengl != NULL) {
            wlOpengl->baseOpengl->draw();
        }
    }
}


void callback_SurfaceDestory(void *ctx) {
    LmOpengl *wlOpengl = static_cast<LmOpengl *>(ctx);
    if (wlOpengl != NULL) {
        if (wlOpengl->baseOpengl != NULL) {
            wlOpengl->baseOpengl->destroy();
        }
    }
}

LmOpengl::LmOpengl() {
}

LmOpengl::~LmOpengl() {
}

void LmOpengl::onCreateSurface(JNIEnv *env, jobject surface) {

    nativeWindow = ANativeWindow_fromSurface(env, surface);
    eglThread = new LmEglThread();
    eglThread->setRenderType(OPENGL_RENDER_HANDLE);
    eglThread->callBackOnCreate(callback_SurfaceCrete, this);
    eglThread->callBackOnChange(callback_SurfacChange, this);
    eglThread->callBackOnDraw(callback_SurfaceDraw, this);

    baseOpengl = new LmPlayYUV();
    eglThread->onSurfaceCreate(nativeWindow);
}

void LmOpengl::onChangeSurface(int width, int height) {
    if (eglThread != NULL) {
        LOGE("width %d height %d", width, height);
        if (baseOpengl != NULL) {
            baseOpengl->surface_width = width;
            baseOpengl->surface_height = height;
        }
        eglThread->onSurfaceChange(width, height);
    }
}


void LmOpengl::onDestorySurface() {
    if (eglThread != NULL) {
        eglThread->destroy();
    }
    if (baseOpengl != NULL) {
        baseOpengl->destorySorce();
        delete baseOpengl;
        baseOpengl = NULL;
    }
    if (nativeWindow != NULL) {
        ANativeWindow_release(nativeWindow);
        nativeWindow = NULL;
    }

    if (pilex != NULL) {
        free(pilex);
        pilex = NULL;
    }
}

void LmOpengl::setYuvData(void *y, void *u, void *v, int w, int h) {
    if (baseOpengl != NULL) {
        baseOpengl->setYuvData(y, u, v, w, h);
    }
    if (eglThread != NULL) {
        eglThread->notifyRender();
    }
}
```

LmPlayYUV.h

```c++
#include "LmBaseOpengl.h"
#include "../matrix/LmMatrixUtil.h"
#include "../util/LmShaderUtil.h"

class LmPlayYUV : public LmBaseOpengl{

public:
    GLint vPosition;
    GLint fPosition;
    GLint u_matrix;

    GLint sampler_y;
    GLint sampler_u;
    GLint sampler_v;

    GLuint samplers[3];


    float matrix[16];
    void *y = NULL;
    void *u = NULL;
    void *v = NULL;
    int yuv_wdith = 0;
    int yuv_height = 0;

public:
    LmPlayYUV();

    ~LmPlayYUV();

    void onCreate();

    void onChange(int w, int h);

    void draw();

    void destroy();

    void destorySorce();

    void setMatrix(int width, int height);

    void setYuvData(void *y, void *u, void *v, int width, int height);
};
```

```c++
#include "LmPlayYUV.h"

LmPlayYUV::LmPlayYUV() {
}

LmPlayYUV::~LmPlayYUV() {
}

void LmPlayYUV::onCreate() {
    vertex = "attribute vec4 v_Position;\n"
             "attribute vec2 f_Position;\n"
             "varying vec2 ft_Position;\n"
             "uniform mat4 u_Matrix;\n"
             "void main() {\n"
             "    ft_Position = f_Position;\n"
             "    gl_Position = v_Position * u_Matrix;\n"
             "}";

    fragment = "precision mediump float;\n"
               "varying vec2 ft_Position;\n"
               "uniform sampler2D sampler_y;\n"
               "uniform sampler2D sampler_u;\n"
               "uniform sampler2D sampler_v;\n"
               "void main() {\n"
               "   float y,u,v;\n"
               "   y = texture2D(sampler_y,ft_Position).r;\n"
               "   u = texture2D(sampler_u,ft_Position).r - 0.5;\n"
               "   v = texture2D(sampler_v,ft_Position).r - 0.5;\n"
               "\n"
               "   vec3 rgb;\n"
               "   rgb.r = y + 1.403 * v;\n"
               "   rgb.g = y - 0.344 * u - 0.714 * v;\n"
               "   rgb.b = y + 1.770 * u;\n"
               "\n"
               "   gl_FragColor = vec4(rgb,1);\n"
               "}";

    program = createProgrm(vertex, fragment, &vShader, &fShader);
    LOGD("opengl program is %d %d %d", program, vShader, fShader);
    vPosition = glGetAttribLocation(program, "v_Position");//顶点坐标
    fPosition = glGetAttribLocation(program, "f_Position");//纹理坐标
    sampler_y = glGetUniformLocation(program, "sampler_y");
    sampler_u = glGetUniformLocation(program, "sampler_u");
    sampler_v = glGetUniformLocation(program, "sampler_v");
    u_matrix = glGetUniformLocation(program, "u_Matrix");

    glGenTextures(3, samplers);

    for (int i = 0; i < 3; i++) {
        glBindTexture(GL_TEXTURE_2D, samplers[i]);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
        glBindTexture(GL_TEXTURE_2D, 0);
    }
}

void LmPlayYUV::onChange(int width, int height) {
    surface_width = width;
    surface_height = height;
    glViewport(0, 0, width, height);
    setMatrix(width, height);
}

void LmPlayYUV::draw() {
    glClearColor(0.1f, 0.1f, 0.5f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT);
    glUseProgram(program);

    glUniformMatrix4fv(u_matrix, 1, GL_FALSE, matrix);

    glEnableVertexAttribArray(vPosition);
    glVertexAttribPointer(vPosition, 2, GL_FLOAT, false, 8, vertexs);
    glEnableVertexAttribArray(fPosition);
    glVertexAttribPointer(fPosition, 2, GL_FLOAT, false, 8, fragments);

    if (yuv_wdith > 0 && yuv_height > 0) {
        if (y != NULL) {
            glActiveTexture(GL_TEXTURE0);
            glBindTexture(GL_TEXTURE_2D, samplers[0]);
            glTexImage2D(GL_TEXTURE_2D, 0, GL_LUMINANCE, yuv_wdith, yuv_height, 0,
                   GL_LUMINANCE, GL_UNSIGNED_BYTE, y);
            glUniform1i(sampler_y, 0);
        }
        if (u != NULL) {
            glActiveTexture(GL_TEXTURE1);
            glBindTexture(GL_TEXTURE_2D, samplers[1]);
            glTexImage2D(GL_TEXTURE_2D, 0, GL_LUMINANCE, yuv_wdith / 2, yuv_height / 2, 0,
                         GL_LUMINANCE, GL_UNSIGNED_BYTE, u);
            glUniform1i(sampler_u, 1);
        }

        if (v != NULL) {
            glActiveTexture(GL_TEXTURE2);
            glBindTexture(GL_TEXTURE_2D, samplers[2]);
            glTexImage2D(GL_TEXTURE_2D, 0, GL_LUMINANCE, yuv_wdith / 2, yuv_height / 2, 0,
                         GL_LUMINANCE, GL_UNSIGNED_BYTE, v);
            glUniform1i(sampler_v, 2);
        }
        glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);
        glBindTexture(GL_TEXTURE_2D, 0);
    }
}

void LmPlayYUV::destroy() {
    glDeleteTextures(3, samplers);
    glDetachShader(program, vShader);
    glDetachShader(program, fShader);
    glDeleteShader(vShader);
    glDeleteShader(fShader);
    glDeleteProgram(program);
}

void LmPlayYUV::destorySorce() {
    yuv_wdith = 0;
    yuv_height = 0;

    if (y != NULL) {
        free(y);
        y = NULL;
    }
    if (u != NULL) {
        free(u);
        u = NULL;
    }
    if (v != NULL) {
        free(v);
        v = NULL;
    }
}

void LmPlayYUV::setMatrix(int width, int height) {
    initMatrix(matrix);
    if (yuv_wdith > 0 && yuv_height > 0) {
        float screen_r = 1.0 * width / height;
        float picture_r = 1.0 * yuv_wdith / yuv_height;

        if (screen_r > picture_r) { //图片宽度缩放
            float r = width / (1.0 * height / yuv_height * yuv_wdith);
            orthoM(-r, r, -1, 1, matrix);
        } else {	// 图片高度缩放
            float r = height / (1.0 * width / yuv_wdith * yuv_height);
            orthoM(-1, 1, -r, r, matrix);
        }
    }
}

void LmPlayYUV::setYuvData(void *Y, void *U, void *V, int width, int height) {
    if (width > 0 && height > 0) {
        if (yuv_wdith != width || yuv_height != height) {
            yuv_wdith = width;
            yuv_height = height;

            if (y != NULL) {
                free(y);
                y = NULL;
            }
            if (u != NULL) {
                free(u);
                u = NULL;
            }
            if (v != NULL) {
                free(v);
                v = NULL;
            }
            y = malloc(yuv_wdith * yuv_height);
            u = malloc(yuv_wdith * yuv_height / 4);
            v = malloc(yuv_wdith * yuv_height / 4);
            setMatrix(surface_width, surface_height);
        }

        memcpy(y, Y, yuv_wdith * yuv_height);
        memcpy(u, U, yuv_wdith * yuv_height / 4);
        memcpy(v, V, yuv_wdith * yuv_height / 4);
    }
}
```



