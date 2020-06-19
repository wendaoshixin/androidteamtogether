# https证书固定
### 1.概述
在公共网络中我们使用安全的SSL/TLS通信协议进行通信，并且使用数字证书来提供加密和认证。HTTPS的握手环节仍然面临（MIM中间人）攻击的可能性，因此CA证书签发机构也存在被黑客入侵的可能性，同时移动设备也面临内置证书被篡改的风险。

### 2.证书锁定原理
证书锁定（SSL/TLS Pinning）提供了两种锁定方式：

Certificate Pinning，证书锁定
Public Key Pinning，公钥锁定
- 2.1 证书锁定
具体操作：将APP代码内置仅接受指定域名的证书，而不接受操作系统或者浏览器内置的CA根证书对应的任何证书。

缺点：CA签发证书存在有效期问题，在证书续期后需要将证书重新内置到APP内。

- 2.2 公钥锁定
具体做法：公钥锁定是提前证书中的公钥并内置到移动端APP内，通过与服务器对比公钥值来验证连接的合法性。
优点：在制作证书密钥时，公钥在证书续期前后可以保持不变（即密钥对不变），所以可以避免证书有效期问题。
### 3.Android中的实现方式

#### 1.  TrustManager
TrustManager是一个比较老的证书锁定方法，主要用于早期的Android版本或者用于一些CA根机构在Android系统中缺失根证书的情形下，当然也适用于自签名证书的锁定
```java
  private final OkHttpClient client;

  public CustomTrust() {
    X509TrustManager trustManager;
    SSLSocketFactory sslSocketFactory;
    try {
      trustManager = trustManagerForCertificates(trustedCertificatesInputStream());
      SSLContext sslContext = SSLContext.getInstance("TLS");
      sslContext.init(null, new TrustManager[] { trustManager }, null);
      sslSocketFactory = sslContext.getSocketFactory();
    } catch (GeneralSecurityException e) {
      throw new RuntimeException(e);
    }

    client = new OkHttpClient.Builder()
        .sslSocketFactory(sslSocketFactory, trustManager)
        .build();
  }

  public void run() throws Exception {
    Request request = new Request.Builder()
        .url("https://publicobject.com/helloworld.txt")
        .build();

    Response response = client.newCall(request).execute();
    System.out.println(response.body().string());
  }

  private InputStream trustedCertificatesInputStream() {
    ... // Full source omitted. See sample.
  }

  public SSLContext sslContextForTrustedCertificates(InputStream in) {
    ... // Full source omitted. See sample.
  }
```
#### 2. 网络安全性设置（[官方文档](https://developer.android.google.cn/training/articles/security-config "官方文档")）
本方案是官方提供，但需要依赖Android N（Android 7.0 API 24）及以后版本，可在APP开发阶段在APP中内置安全性设置，以达到防止中间人攻击的目的，此方法只限制在Android 7.0 API 24以后版本

- 添加网络安全配置文件
```xml
    <?xml version="1.0" encoding="utf-8"?>
    <manifest ... >
        <application android:networkSecurityConfig="@xml/network_security_config"
                        ... >
            ...
        </application>
    </manifest>
```
- 配置自定义 CA
```xml
  <?xml version="1.0" encoding="utf-8"?>
    <network-security-config>

	 	<!--domain-config针对特定网域 -->
		 <!--cleartextTrafficPermitted是否停用http,9以后默认false  -->
        <domain-config cleartextTrafficPermitted="false"> 
		    <!--domain 限定信任的域名  -->
            <domain includeSubdomains="true">secure.example.com</domain>
            <!-- includeSubdomains如果为 "true"，此网域规则将与相应网域及所有子网域（包括子网域的子网域）匹配。否则，该规则仅适用于精确匹配项。-->
            <domain includeSubdomains="true">cdn.example.com</domain>
            <trust-anchors>
			    <!-- 限定信任的证书，添加有效7.0以前默认system和user，7.0以后默认system  -->
			    <certificates src="system" /><!-- 系统内置  -->
                <certificates src="user" />  <!-- 用户添加  -->
                <certificates src="@raw/trusted_roots"/> <!-- 自定义其他  -->
            </trust-anchors>
		   <pin-set expiration="2018-01-01">
		        <!-- 公钥SHA-256   -->
                <pin digest="SHA-256">7HIpactkIAq2Y49orFOOQKurWxmmSFZhBCoQYcRhJ3Y=</pin>
                <!--备用 公钥SHA-256-->
                <pin digest="SHA-256">fwza0LRMXouZHRC8Ei+4PyuldPDcf3UKgO/04cDM1oE=</pin>
           </pin-set>
        </domain-config>
		
	<!--不在 domain-config 涵盖范围内的所有连接所使用的默认配置 -->
	    <base-config cleartextTrafficPermitted="false">
       		 <trust-anchors>
           		 <certificates src="system" />
       		 </trust-anchors>
        </base-config>
    </network-security-config>
```

#### 3.okhttp Certificate Pinning

```java
  private final OkHttpClient client = new OkHttpClient.Builder()
      .certificatePinner(
          new CertificatePinner.Builder()
              .add("publicobject.com", "sha256/afwiKY3RxoMmLkuRW1l7QsPZTJPwDS2pdDROQjXw8ig=")
              .build())
      .build();

  public void run() throws Exception {
    Request request = new Request.Builder()
        .url("https://publicobject.com/robots.txt")
        .build();

    try (Response response = client.newCall(request).execute()) {
      if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

      for (Certificate certificate : response.handshake().peerCertificates()) {
        System.out.println(CertificatePinner.pin(certificate));
      }
    }
  }
```
配置方法：
1.随便设置一个sha256的值访问配置好的https

```java
String hostname = "publicobject.com";
CertificatePinner certificatePinner = new CertificatePinner.Builder()
    .add(hostname, "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")//hostname多的话可以用通配符
    .build();
OkHttpClient client = OkHttpClient.Builder()
    .certificatePinner(certificatePinner)
    .build();

Request request = new Request.Builder()
    .url("https://" + hostname)
    .build();
client.newCall(request).execute();

```
[hostname通配符官方文档](https://square.github.io/okhttp/4.x/okhttp/okhttp3/-certificate-pinner/ "hostname通配符官方文档")

2.报出异常信息
```java
 javax.net.ssl.SSLPeerUnverifiedException: Certificate pinning failure!
Peer certificate chain:
    sha256/afwiKY3RxoMmLkuRW1l7QsPZTJPwDS2pdDROQjXw8ig=: CN=publicobject.com, OU=PositiveSSL
    sha256/klO23nT2ehFDXCfx3eHTDRESMz3asj1muO+4aIdjiuY=: CN=COMODO RSA Secure Server CA
    sha256/grX4Ta9HpZx6tSHkmCrvpApTQGo67CYDnvprLg5yRME=: CN=COMODO RSA Certification Authority
    sha256/lCppFqbkrlJ3EcVFAkeip0+44VaoJUymbnOaEUk7tEU=: CN=AddTrust External CA Root
Pinned certificates for publicobject.com:
    sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=
  at okhttp3.CertificatePinner.check(CertificatePinner.java)
  at okhttp3.Connection.upgradeToTls(Connection.java)
  at okhttp3.Connection.connect(Connection.java)
  at okhttp3.Connection.connectAndSetOwner(Connection.java)
```
3.将异常的公钥哈希粘贴到证书配置中
```java
CertificatePinner certificatePinner = new CertificatePinner.Builder()
    .add("publicobject.com", "sha256/afwiKY3RxoMmLkuRW1l7QsPZTJPwDS2pdDROQjXw8ig=")
    .add("publicobject.com", "sha256/klO23nT2ehFDXCfx3eHTDRESMz3asj1muO+4aIdjiuY=")
    .add("publicobject.com", "sha256/grX4Ta9HpZx6tSHkmCrvpApTQGo67CYDnvprLg5yRME=")
    .add("publicobject.com", "sha256/lCppFqbkrlJ3EcVFAkeip0+44VaoJUymbnOaEUk7tEU=")
    .build();
```