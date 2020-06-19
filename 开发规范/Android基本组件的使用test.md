# Android基本组件的使用



## Activity

**定义：**用户和Android设备进行交互的可视化窗口界面.

**1.启动模式：**在AndroidManifest的清单文件中，使用android:launchMode标签配置

 -  standard:标准模式，此模式的activity会进入到调用者的任务栈中(singleInstance的调用者除外).
 -  singleTask:单任务栈单例模式，此模式的activity在单任务栈中是可复用的.
 -  singleTop:单任务栈栈顶单例模式，此模式的activity在单任务栈的栈顶时是可复用的.
 -  singleInstance.单任务栈，任务栈中只有一个该模式的activity

**2.任务栈相关属性：**

 -  configChanges:可以检测界面属性的变化，例如尺寸的、方向、键盘事件等 ,在activity中对应的回调方法是onConfigurationChanged方法，配置该属性后，可以优化activity的生命周期.
 -  例如横竖屏切换时：配置“orientation|keyboardHidden|screenSize”,activity不会被销毁,不会重新走其生命周期方法.
 -  taskAffinity:默认的taskAffinity是应用的包名.当activity要配置taskAffinity的时候，需要以 . 开头，后边加上逻辑路径名，例如 android:taskAffinity=".pay.orderdetail"
 -  flag:在启动activity时，通过给Intent设置相关Flag，会改变activity启动模式的行为，例如Intent.FLAG_ACTIVITY_NEW_DOCUMENT,Intent.
FLAG_ACTIVITY_MULTIPLE_TASK,Intent.FLAG_ACTIVITY_NEW_TASK等.

 举例：

​     **1).** standard模式的activity，在启动时，加上addFlags(Intent.FLAG_ACTIVITY_NEW_DOCUMENT | Intent.FLAG_ACTIVITY_NEW_TASK)，会启动在在一个新的任务栈中.

​    **2).** singleTask模式的activity，通过配置taskAffinity，会启动在一个新的任务栈中.在手机上的表现是最近应用列表中，同一个应用会有两个页面.

**3.Activity启动：**

 -  activity可以通过 startActivity(intent)、startActivityForResult(intent，requestCode)两种方式启动，通过后者启动后，被启动的activity可以通过setResult(requestCode,intent)方法进行结果通知.
 -  （建议）启动封装：将启动activity的参数封装成一个静态方法，在调用处直接调用该静态方法.





 ## Service

 **定义：**运行于应用后台的服务，长时间运行，不需要和用户进行交互（WindowManager情况除外）;

 **1.启动方式:** 

 ```java
	      1).context.startService(intent)
	
	      2).bindService(intent, new ServiceConnection() {
	         @Override
	         public void onServiceConnected(ComponentName name, IBinder service) {
	             LocalBinder binder = (LocalBinder) service; // LocalBinder.Stub.asInterface(service);
	         }
	
	         @Override
	         public void onServiceDisconnected(ComponentName name) {
	         }
	     },BIND_AUTO_CREATE); 
 ```

 **2.断开重连：**

 ```java
	     1)可以在onServiceDisconnected中重连
	 		
	 		 2)可以通过
	      binder.linkToDeath(new IBinder.DeathRecipient() {
	           @Override
	           public void binderDied() {
	             // 重连
	           }
	      },0);
 ```

 **3.AIDL:**

 ```java
	    1)支持的数据类型:
	 				1>> java 的 8 种数据类型：byte、short、int、long、float、double、boolean、char
	 				2>> String、charSequence、List、Map。其中List和Map中的元素 AIDL支持的数据类型.
	 				3>> 自定义数据类型
            
	 		2）数据容量：在Binder底层容量显示在4M左右，在App层超过1M的数据，不要通过AIDL传输.
	 		
	 		3) 自定义数据类型：
			 	 定义BeanName.aidl
	 			 package com.demo.pacakgeName;
	 			 parcelable BeanName;
	 			 
	 			 定义Binder或接口类aidl文件
	 			 package com.demo.pacakgeName;
	 			 import com.demo.pacakgeName.BeanName;
	 			 interface OnStateChangeListener {
	 			 		void onBeanStateChanged(in BeanName beanName);
	 			 }
	 			 
	 			 实现Bean、Binder、接口等java文件：
	 			 举例：
	 			 	public class BeanName implements Parcelable {
	 			 	   public BeanName(Parcel in) {
				 			 	   in.readInt();
				 			 	   //....
	 			 	   }
	 			 	   
	 			 	   @Override
	 			 	   public void writeToParcel(Parcel dest,int flags){
	 			 	   		   dest.writeInt(age);
	 			 	   		   //....
	 			 	   }
	 			 	   
	 			 	   public static final Creator<BeanName> CREATOR = new Creator<BeanName>(){
	              @Override
	              public BeanName createFromParcel(Parcel source) {
	              	return new BeanName(source);
	              }
	              @Override
	              public BeanName[] newArray(int size) {
	              	return new BeanName[size];
	              }
	              
	 			 	   }
	 			 	}
 ```



**4.IntentService:**

​     内部封装了HandlerThread，以及对应的Handler，startService(intent)被传递到了线程的消息队列中等待被串行调度，被调度后，回调到handleIntent(intent)方法.



**5.Messenger的使用：**

```java
	 		1）在Service中定义服务端Messenger对象：
	 			private Messenger mMessenger = new Messenger(new Handler()
	         {
	            @Override
	            public void handleMessage(Message msgfromClient)
	            {
	                Message msgToClient = Message.obtain(msgfromClient);
	                switch (msgfromClient.what)
	                {
	                    case MSG_SUM:
	                        msgToClient.what = MSG_SUM;
	                        msgToClient.arg2 = msgfromClient.arg1 + msgfromClient.arg2;
	                        msgfromClient.replyTo.send(msgToClient);ak;
	                }
	                super.handleMessage(msgfromClient);
	            }
	        });
	        
	     2）在Service的onBinder中，返回Messenger的binder对象，mMessenger.getBinder();
	    
	     3）在Activity中定义两个客户端Messenger对象,一个用于发送，一个用于回调,并在onServiceConnected中初始化：
	    			private Messenger mMessenger = new Messenger(new Handler()
	          {
	              @Override
	              public void handleMessage(Message msgFromServer)
	              {
	                  switch (msgFromServer.what)
	                  {
	                      case MSG_SUM:
	                          break;
	                  }
	                  super.handleMessage(msgFromServer);
	              }
	          });
	          private Messenger mServiceMessenger;
	          private ServiceConnection mConn = new ServiceConnection()
	          {
	              @Override
	              public void onServiceConnected(ComponentName name, IBinder service)
	              {
	                  mServiceMessenger = new Messenger(service);
	              }
	
	              @Override
	              public void onServiceDisconnected(ComponentName name)
	              {}
	          };
	          
	      4）发送消息：
	     			Message msgFromClient = Message.obtain();
	     			msgFromClient.replyTo = mMessenger;
	     			mServiceMessenger.send(msgFromClient);
```





## BroadcastReceiver

  **定义：**广播组件之间传递数据的一种机制，这些组件可以位于不同的进程中，起到进程间通信的作用

   -  **1.注册方式：**都可用于跨进程通信

```java
静态注册：在AndroidManifest中注册，在intent-filter中配置action、priority等属性

动态注册：在代码程序中注册.在IntentFilter对象中，通过setAction、setPriority方法设置属性
```

 - **2.广播接受者类型**

    普通广播、系统广播、有序广播、粘性广播、本地广播

   **1)普通广播**：开发者自定义的广播，用于应用内、应用之间数据传递.
       		
   **2)系统广播**：用于接受系统事件、系统状态变化等，部分权限需要权限声明，例如监听系统开机事件、网络状态变化、电量过低等.

   

   **3)有序广播**：广播接受者通过定义优先级priority属性，进行有序的接受，priority的定义范围一般在-1000～1000，数值越大优先级越高.有序广播通过sendOrderedBroadcast方法发送，接受者可以通过setResultExtras(bundle)方法添加数据到传递链中，也可以通过abortBroadcast终止广播的向下传送.

   

   **4)粘性广播（强制）**：不推荐使用，在API某些版本中已经废弃掉。

   

   **5)本地广播（应用内广播)**: 用于单进程、应用内的数据传递，基于Handler消息机制实现的广播，通过LocalBroadcastManager来注册receiver和发送广播.

  **使用举例：**

```java
LocalBroadcastManager.getInstance(this).registerReceiver(receiver);
LocalBroadcastManager.getInstance(this).sendBroadcast(intent);
LocalBroadcastManager.getInstance(this).unregisterReceiver(receiver);
```







## ContentProvider

 **定义：**Android中提供的专门用于不同应用间数据交互和共享的组件,一般用于SQLiteOpenHelper的进一步封装.

  **1.URI:**
	 	content://com.android.contacts/contacts/contactId
	 	schema: 在Android中固定为content://
	      authority: 用于唯一标识一个ContentProvider。
	      path: ContentProvider中数据表的表名。
	      id: 数据表中数据的标识，可选字段。
	      
	      
  **2.UriMatcher: **

​			帮助匹配ContentProvider中的Uri,提供了两个方法——addURI和match方法
​	  		使用举例：

```java
	  				UriMatcher matcher = new UriMatcher(UriMatcher.NO_MATCH);
​	  				String AUTHORITY = "com.android.demo.demo";
​	  				int tablename_uri_code = 0;
​	  				matcher.addURI(AUTHORITY,"tablename",tablename_uri_code);
​	  				int uri_code = matcher.match(uri);
```

**3.ContentUris：**

​			Uri的辅助类，提供了withAppendedId、parseId、appendId三个静态方法.
​	  		使用举例：
​	  			

```java
	 					 Uri uri = Uri.parse("content://com.android.demo/demo");
	  				 Uri withAppendedIdUri = ContentUris.withAppendedId(uri, 1);//content://com.android.demo/demo/1
	  				 long parseId = ContentUris.parseId(withAppendedIdUri);
```

 **4.ContentProvider的定义**：

​			继承android.content.ContentProvider,并实现其中的以下方法：
​	  		onCreate: 在该方法中初始化数据操作对象，例如DBOpenHelper、SQLiteDatabase等工具类.
​	  		query、insert、delete、update等数据库方法.
​	  		
​	  		

 **5.ContentProvider的声明：**
	

```html
      <provider
	             android:name=".MContentProvider"
	             android:authorities="com.android.demo.demo"
	             android:readPermission="com.android.demo.demo.READ_PERMISSION"
	             android:writePermission="com.android.demo.demo.WRITE_PERMISSION"
	             android:process=":provider"
	             android:exported="true"/>
```

​	             

**6.ContentResolver:**

​		数据使用端的API，通过contextWrapper.getContentResolver()获取对象.
​	  		使用举例：

```java
	  			String AUTHORITY = "com.android.demo.demo";
​	  			Uri TABLE_URI = Uri.parse("content://" + AUTHORITY + "/tableName");
​	  			ContentValues contentValues = new ContentValues();
​	  			contentValues.put("id",0);
​	              contentValues.put("name","peter");
​	  			context.getContentResolver().insert(TABLE_URI,contentValues);
​	  			context.getContentResolver().update(TABLE_URI,contentValues,
​	  			                                    where/**"id = ?"**/, 
​	  			                                    selectionArgs/**new String[] {"0"}**/);
​	  			context.getContentResolver().query(TABLE_URI,projection,selection,selectionArgs,sortOrder);
​	  		    context.getContentResolver().delete(TABLE_URI,where,selectionArgs)；
```





**7.ContentObserver:**

​	    用于监听ContentProvider变化的抽象类，通过ContentResolver的registerContentObserver和unregisterContentObserver方法来注册和注销ContentObserver监听器.
​	   			
使用举例：

```java
ContentObserver contentObserver = new ContentObserver(mHandler) {
	            @Override
	            public void onChange(boolean change, Uri uri) {
	                super.onChange(selfChange, uri);
	                
	              }
	          };
context.getContentResolver().registerContentObserver(AUTHORITY,true,contentObserver);
context.getContentResolver().unregisterContentObserver(contentObserver);
```

​	          







## Handler

**定义：用于线程间消息通信.**

**1:常用的代码写法：**

```java
1）Handler.Callback callback = new Handler.Callback() {
	                  @Override
	                  public boolean handleMessage(@NonNull Message msg) {
	                      return false;
	                  }
	              };
	           Handler mHandler = new Handler(callback);
	     
	        2）Handler mHandler = new Handler() {
	                  @Override
	                  public void handleMessage(@NonNull Message msg) {
	                      super.handleMessage(msg);
	                  }
	              };
	        3）private static class CustomHandler extends Handler{  
	
	              private final WeakReference<Activity> mActivty;  
	
	              private CustomHandler(Activity mActivty) {  
	                  this.mActivty = new WeakReference<HandlerActivity>(mActivty);  
	              }  
	
	              @Override  
	              public void handleMessage(Message msg) {  
	                  super.handleMessage(msg);  
	                  HandlerActivity activity = mActivty.get();  
	                  if (activity != null) {
	                  }  
	              }  
	          } 

```

 

**2:发送消息的常用方法:** 以下方法相对应也有post方法

 ```java
 sendMessage(null);
 sendMessageDelayed(null, 0);
 sendMessageAtFrontOfQueue(null);
 sendMessageAtTime(null, 0);
 ```



 **3.同步屏障、异步消息：**

 Handler中消息分为同步消息、异步消息，通过内部属性mAsynchronous来区分，

  - **1）异步消息通过以下两种方式获取**：

    ​    一: 创建用于发送异步消息的Handler

    ```java
    	public Handler(boolean async);
      public Handler(Callback callback, boolean async);
      public Handler(Looper looper, Callback callback, boolean async);
    ```

    ​    二：调用Message的方法，将消息标识为异步消息.

    ```java
    setAsynchronous(true)
    ```

    

    

    

   - **2)同步屏障：同步屏障的添加和移除是通过反射调用**

   ```java
MessageQueue的 postSyncBarrier和removeSyncBarrier方法：
	    	 示例：
	    	 		// 添加屏障
	          MessageQueue queue=mHandler1.getLooper().getQueue();
	          Method method=MessageQueue.class.getDeclaredMethod("postSyncBarrier");
	          int token= (int) method.invoke(queue);
	          
	          // 移除屏障
	          MessageQueue queue=mHandler1.getLooper().getQueue();
	          Method method=MessageQueue.class.getDeclaredMethod("removeSyncBarrier",int.class);
	          method.invoke(queue,token);
   ```

 **4.注意事项：**

  	**1)** 采用匿名内部类的方式来处理消息，一定要在宿主生命周期结束时，移除所有的消息和任务,否则会存在内存泄漏情况.

   	**2)** 添加同步屏障后，一定要发送异步消息，否则之前的同步消息会一直等待，无法被执行.