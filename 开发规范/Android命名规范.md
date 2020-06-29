# Android命名规范

##### 一、标识符

1、小驼峰：方法 allFun

2、大驼峰：类名SplashActivity、IPhoneHelper

3、下划线：xml文件 activity_splash

##### 二、包名规范

<u>*com.leimans.book.ui*</u>

采用如下规则：【com】.【公司名/组织名】.【项目名称】.【模块名】,然后在这个目录下根据业务逻辑进行分层

|                                     |                                                              |
| ----------------------------------- | ------------------------------------------------------------ |
| com.example.app.activitys           | 用来组织Activity类                                           |
| com.example.app.base                | 基础共享的类，如多个Activity共享的 BaseActivity或整个应用共享的MyApplication类 |
| com.example.app.adapter             | 项目中用到的适配器类                                         |
| com.example.app.view                | 自定义的View，如常用的TitleBarView                           |
| com.example.app.util                | 工具类，如HttpUtil，ImageUtil，FileUtil                      |
| com.example.app.db                  | 数据库类，如DataBaseHelper，MessageDB                        |
| com.example.app.service             | 服务类，如GetMsgService                                      |
| com.example.app.constant            | 常量类                                                       |
| com.example.app.domain/modle/entity | 元素实体类，如对应注册用户信息的User类， 对应聊天信息的TextMessage类 |
| com.example.app.broadcast           | 广播服务类                                                   |

###### 补充：

1、utils（工具类）、adapter（适配器）、base（基类）、bean（对象类）、config（配置类）、httpservice（请求接口）、interface（操作接口类）、model（M继承interface）、view、widget(自定义视图)、db、service、broadcast、provider



