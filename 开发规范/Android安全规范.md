# Android 安全规范

##### 1.【规范要求】【推荐】

针对不需要进行跨应用调用的组件，应在配置文件（AndroidManifest.xml）中显示配置 android:exported=”false”属性。
【详细说明】
组件配置 android:exported=”false”属性，表明它为私有组件，只可在同一个应用程序组件间或带有相同用户 ID 的应用程序间才能启动或绑定该服务。在非必要情况下，如果该属性设置为“true”，则该组件可以被任意应用执行启动操作，造成组件恶意调用等风险。

##### 2.【规范要求】【推荐】
因特殊需要而公开的 Activity、Service、Broadcast Receiver、Content Provider 组件建议添加自定义 permission 权限进行访问控制。
##### 【详情说明】
因特殊需要而公开（exported=”true”）的对于需要公开的 Activity、Service、Broadcast Receiver、Content Provider 组件采用自定义访问权限的方法提供访问控制波保护。通过自定义访问权限保护后公开的组件只能被申请了该权限的外部应用（Application）调用，未申请权限的外部应用（Application）在调用时将会出现“java.lang.SecurityException: Not allowed to bind to service Intent”异常，造成调用失败。
程序中的启动/使用 Activity、Service、Broadcast Receiver、Content Provider 则由程序定位与需求来确认是否添加自定义访问权限，启动的 Activity、Service、Broadcast Receiver、Content Provider 本身是需要被其他程序进行调用的，如果没有特殊需求（该程序只允许指定 APP 启动）的话就不能添加该权限。

##### 3.【规范要求】【推荐】
应避免使用隐式调用 Intent ，包括 Activity、Content provider、Broadcast receiver、Service 等，为了数据安全与性能消耗须使用显式调用尽量减少使用隐式调用。

##### 4.【规范要求】【强制】
开放的 activity/service/receiver 等需要对传入的 intent 做合法性校验。

##### 5.【规范要求】【推荐】
使用 PendingIntent 时，禁止使用空 intent，同时禁止使用隐式 Intent
##### 【详情说明】

使用 PendingIntent 时，使用了空 Intent,会导致恶意用户劫持修改 Intent 的内容。禁止使用一个空 Intent 去构造 PendingIntent，构造 PendingIntent 的 Intent一定要设置 ComponentName 或者 action。
PendingIntent 可以让其他 APP 中的代码像是运行自己 APP 中。PendingIntent的intent接收方在使用该intent时与发送方有相同的权限。在使用PendingIntent时，PendingIntent 中包装的 intent 如果是隐式的 Intent，容易遭到劫持，导致信息泄露。
正例：
```java

Intent intent = new Intent(this, SomeActivity.class);
PendingIntent pendingIntent = PendingIntent.getActivity(this, 1, intent, PendingIntent.FLAG_UPDATE_CURRENT);
try {
    pendingIntent.send();
} catch (PendingIntent.CanceledException e) {
    e.printStackTrace();
}
```

反例 1：

```java
Bundle addAccountOptions = new Bundle();
mPendingIntent = PendingTntent.getBroadcast(this, 0, new Intent, 0);
addAccountOptions.putParcelable(KEY_CALLER_IDENTITY, mPendingIntent);
addAccountOptions.putBoolean(EXTRA_HAS_MULTIPLE_USERS,
Utils.hasMultipleUsers(this));
AccountManager.get(this).addAccount(accountType,
    null,
    null,
    addAccountOptions,
    null,
    mCallback,
    null);
```

反例 2：
mPendingIntent 是通过 new Intent()构造原始 Intent 的，所以为“双无”Intent，这个PendingIntent最终被通过AccountManager.addAccount 方法传递给了恶意APP接口。

```java
Intent intent = new Intent("com.test.test.pushservice.action.METHOD");
intent.addFlags(32);
intent.putExtra("app",msg);
PendingIntent.getBroadcast(this, 0, intent, 0));
```

如上代码PendingIntent.getBroadcast，PendingItent中包含的Intent为隐式intent，因此当 PendingIntent 触发执行时，发送的 intent 很可能被嗅探或者劫持，导致 intent 内容泄漏。

##### 6.【规范要求】【推荐】
在实现的 HostnameVerifier 子类中，需要使用 verify 函数效验服务器主机名的合法性，否则会导致恶意程序利用中间人攻击绕过主机名效验。
##### 【详情说明】
在握手期间，如果 URL 的主机名和服务器的标识主机名不匹配，则验证机制可以回调此接口的实现程序来确定是否应该允许此连接。如果回调内实现不恰当，默认接受所有域名，则有安全风险。
反例：

```java
HostnameVerifier hnv = new HostnameVerifier() {
    @Override
    public boolean verify(String hostname, SSLSession session) {
        // 总是返回 true，接受任意域名服务器
        return true;
    }
};
HttpsURLConnection.setDefaultHostnameVerifier(hnv);
```

正例：

```java
HostnameVerifier hnv = new HostnameVerifier() {
    @Override
    public boolean verify(String hostname, SSLSession session) {
        //示例
        if("yourhostname".equals(hostname)){
            return true;
        } else {
            HostnameVerifier hv = HttpsURLConnection.getDefaultHostnameVerifier();
            return hv.verify(hostname, session);
        }
    }
};
```

##### 7.【规范要求】【推荐】
利用 X509TrustManager 子类中的 checkServerTrusted 函数效验服务器端证书的合法性。
##### 【详情说明】
在实现的 X509TrustManager 子类中未对服务端的证书做检验，这样会导致不被信任的证书绕过证书效验机制。
反例：
```java

TrustManager tm = new X509TrustManager() {
    public void checkClientTrusted(X509Certificate[] chain, String authType) throws CertificateException {
        //do nothing，接受任意客户端证书
    }
    public void checkServerTrusted(X509Certificate[] chain, String authType) throws CertificateException {
        //do nothing，接受任意服务端证书
    }
    public X509Certificate[] getAcceptedIssuers() {
        return null;
    }
};
sslContext.init(null, new TrustManager[] { tm }, null);
```
##### 8.【规范要求】【强制】
密钥切勿硬编码到代码中。

##### 9.【规范要求】【推荐】
加密算法：使用不安全的 Hash 算法(MD5/SHA-1)加密信息，存在被破解的风险，建议使用 SHA-256 等安全性更高的 Hash 算法。

##### 10.【规范要求】【推荐】
避免将数据储存到 sdcard 中，尽量使用 sqlite、sharedpreferences 或系统私有目录的 file 文件进行数据储存。
##### 【详情说明】
使用外部存储实现数据持久化，这里的外部存储一般就是指的是 sdcard。使用 sdcard 存储的数据，不限制只有本应用访问，任何可以有访问 Sdcard 权限的应用均可以访问，容易导致信息泄漏安全风险。

##### 11.【规范要求】【推荐】
数据存储在 Sqlite 或者轻量级存储需要对数据进行加密，取出来的时候进行解密。

##### 12.【规范要求】【推荐】
使用安全的 SQL 语句查询方式，避免出现命令拼接的形式
##### 13.【规范要求】【强制】
不要把敏感信息打印到 log 中。

##### 14.【规范要求】【推荐】
Android5.0 以后安全性要求较高的应用应该使用 window.setFlag(LayoutParam.FLAG_SECURE) 禁止录屏。

##### 15.【规范要求】【推荐】
在使用 WebView 控件时，应显示关闭控件自带的记住密码功能。即：设置 WebView.getSettings().setSavePassword(false);
【详情说明】
Google 在设计 WebView 的时候提供默认自带记住密码的功能，即程序在不设置 theWebView.getSettings().setSavePassword(false);的时候 WebView 在使用密码控件后会自动弹出界面提示用户是否记住密码，如果用户选择“记住”选择项后密码会明文储存在/data/data/com.package.name/databases/webview.db 中，如果设备中出现了 Root 提权的其他应用的时候该应用则可直接读取所有应用通过 webView 储存的密码。所以在使用 Webview 时应显示关闭 Webview 的自动保存密码功能，防止用户密码被 Webview 明文存储在设备中。
##### 16.【规范要求】【推荐】
避免 webview 通过 file:schema 方式访问本地敏感数据。
##### 17.【规范要求】【推荐】
正式发布的应用应关闭数据备份功能。应在 AndroidManifest.xml 的 Application 参数设置中将 android:allowBackup 参数显示设置为“false”，关闭非 root 情况下允许对应用数据的备份与恢复功能。【详情说明】
当在 AndroidManifest.xml 中 application 配置参数 allowBackup 被设置为 true 或不设置该标志时，应用程序数据可以再非 root 状态下进行数据的备份和恢复，攻击者可以通过 adb 调试指令直接复制应用程序数据。造成应用数据泄露风险。




