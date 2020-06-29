# Android 安全规范


##### 1.【规范要求】【推荐】
公开的组件建议添加自定义 permission 权限进行访问控制。

##### 2.【规范要求】【强制】
公开组件对外部输入 Intent 数据的合法性做针对性校验
##### 3.【规范要求】【强制】
ContentProvider对外部输入的数据做合法性校验


##### 4.【规范要求】【推荐】
使用 PendingIntent 时，禁止使用空 intent，同时禁止使用隐式 Intent
##### 【详情说明】

使用 PendingIntent 时，使用了空 Intent,会导致恶意用户劫持修改 Intent 的内容。构造 PendingIntent 的 Intent一定要设置 ComponentName 或者 action。
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


```java
Intent intent = new Intent("com.test.test.pushservice.action.METHOD");
intent.addFlags(32);
intent.putExtra("app",msg);
PendingIntent.getBroadcast(this, 0, intent, 0));
```



##### 5.【规范要求】【推荐】
在实现的 HostnameVerifier 子类中，需要使用 verify 函数效验服务器主机名的合法性，否则会导致恶意程序利用中间人攻击绕过主机名效验。

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

##### 6.【规范要求】【推荐】
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
##### 7.【规范要求】【强制】
密钥切勿硬编码到java代码中。

##### 8.【规范要求】【推荐】
避免使用不安全的 Hash 算法(MD5/SHA-1)加密信息，存在被破解的风险，建议使用 SHA-256 等安全性更高的 Hash 算法。

##### 9.【规范要求】【推荐】
避免将敏感数据储存到 sdcard 中，尽量使用 sqlite、sharedpreferences 或系统私有目录的 file 文件进行数据储存。
##### 【详情说明】
使用外部存储实现数据持久化，这里的外部存储一般就是指的是 sdcard。使用 sdcard 存储的数据，不限制只有本应用访问，任何可以有访问 Sdcard 权限的应用均可以访问，容易导致信息泄漏安全风险。

##### 10.【规范要求】【推荐】
数据存储在 Sqlite 或者sharedpreferences需要对数据进行加密，取出来的时候进行解密。


##### 11.【规范要求】【强制】
不要把敏感信息打印到 log 中。

##### 12.【规范要求】【推荐】
Android5.0 以后安全性要求较高的应用应该使用 window.setFlag(LayoutParam.FLAG_SECURE) 禁止录屏。

##### 13.【规范要求】【推荐】
避免 webview 通过 file:schema 方式访问本地敏感数据。
##### 14.【规范要求】【推荐】
对 JS 访问 JavaScript Interface 做权限控制，避免不信任的页面访问敏感的 Interface 接口。



