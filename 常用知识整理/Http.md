# Http
## 为什么要分层？
- 因为网络的不稳定性。

## 网络四层模型
- Application Layer 应⽤用层:HTTP、FTP、DNS 
- Transport Layer 传输层:TCP、UDP 。其中TCP 的连接是有返回的，UDP没有。
- Internet Layer ⽹网络层:IP 传输数据的
- Link Layer 数据链路路层:以太⽹网、Wi-Fi。

## TCP 的3次握手与连接的关闭（关闭需要4次交互）

![](https://user-gold-cdn.xitu.io/2020/1/14/16fa2fa08e184a2c?w=580&h=541&f=png&s=44010)

![](https://user-gold-cdn.xitu.io/2020/1/14/16fa2fa8abc5b8a3?w=571&h=638&f=png&s=50437)

## 长连接的实现方式，以及为什么需要长连接？
- 为什么：当某个 TCP 连接在⼀一段时间不不通信之后，⽹网关会出于⽹网络性能考虑⽽而关闭这条 TCP 连接和公⽹网的连接通道，导致 这个 TCP 端⼝口不不再能收到外部通信消息，即 TCP 连接被动关闭
- 怎么实现：心跳。即在一定间隔时间内，使用 TCP 连接发送超短无意义消息来让网关不能将自己定义为「空闲连 接」，从⽽防止网关将⾃己的连接关闭。

# 加密
## 对称加密
- 定义：双方使用同一个密钥，通过加密算法和解密算法进行加解密。重点是两套算法。
- 经典算法：AES / DES
- 优点：速度快
- 缺点：密钥如果外泄了就可能不安全了。
- 图示

![](https://user-gold-cdn.xitu.io/2020/1/14/16fa2ffeb18e6237?w=797&h=165&f=png&s=22959)

## 非对称加密
- 定义：公钥加密，私钥解密，使用的是同一套算法。
- 作用：加解密数据 或者 可以用来做数字签名
- 怎么用来做数据签名？ ： 使用原数据hash值用私钥进行签名，然后把签名数据、原数据、公钥打包出去，如果谁能生成这个签名数据，就能证明这是谁的，因为私钥在你这，所以只可能是拥有者的。
- 经典算法：RSA（加密，签名都可以） / DSA (仅用于签名，但速度更快)
- 优点：可以把公钥给出去，解决了对称加密无法传输密钥的问题。
- 缺点：计算复杂，慢。
- 加密的图示

![](https://user-gold-cdn.xitu.io/2020/1/14/16fa306aee96dd7c?w=794&h=179&f=png&s=25800)

![](https://user-gold-cdn.xitu.io/2020/1/14/16fa3070031a9b9d?w=792&h=244&f=png&s=31925)

- 签名的图示

![](https://user-gold-cdn.xitu.io/2020/1/14/16fa307545826097?w=794&h=217&f=png&s=40421)
hash过的

![](https://user-gold-cdn.xitu.io/2020/1/14/16fa307d796f0639?w=785&h=298&f=png&s=69601)

## 加密和登陆是有本质区别的，登陆只是校验相不相等，没有数学那些东西

# 编码
- 含义：把数据由一种数据格式转换成另一种数据格式。
## Base64
- Base64: 将原数据每6位对应成Base64索引表中的一个字符排成一个字符串。
- 不可逆！
- Base64后的数据肯定是比原数据要大的。因为原来是8位一个字符，现在是6位一个字符了啊。
- 用处1：扩充了储存和传输的途径，比如可以通过发送文本的形式发送二进制数据了。
- Base64不是加密！！！是个人用点心都能逆向出来，码表在那放着，自己去看啊。
- 有个Base58的变种，去除掉了几个不方便写的字符，区块链用得多。
- urlEncoder是特殊的Base64。暂且这样理解吧。

## 压缩与解压缩
- 举个例子
```
压缩前：fffffffffffffff
压缩后：f*15

大概就是这么个意思
```
- 压缩是编码！

# 序列化
- 目的：让内存中的对象可以被储存和传输
- 含义：把数据由内存中的对象转换成字节序列。然后就可以储存或者传输了。
- 序列化不是编码，不是编码，不是编码！！！

# Hash
- 定义：把任意数据转换成指定范围大小的数据，通常挺小的，256字节以内。
- 作用：从数据中提出摘要信息，所以最主要的用途是做数字签名。想想上面的非对称加密。
- 举个例子
```
把你人做Hash，那么他就通过你的摘要（性别年龄身份证号码）等生成一个唯一的数据。这个数据能证明你是你。
```
- 用途：验证完整性，真实正确性。
- 常用算法： MD5, SHA256。
- Hash不是加密！！！都不可逆加密个屁啊。Hash是单项的，因为只拿了他的摘要啊。有你的身份证号码就知道你多帅？
- Hash不是编码！！！这是不可逆的，兄弟Hash后的数据没办法恢复。同上。
- 实际用途
```
1: 数据完整性
2: 快速查找，HashMap啊
3: 隐私保护：通过加盐的方式，可以储存密码。比如登陆密码啥的。后台不经常这么搞吗？加个盐md5一下。
```

# Https
- 工作在SSL / TLS 上的 Http，就是加密通讯的Http。
- 简单来说就是先使用非对称加密商讨出密钥，然后使用对称加密进行通讯。
- 为什么这样搞？因为非对称加密慢，直接使用对称加密如何给出密钥是个问题。

# OkHttp分析
## 核心是那一串的interceptors，就是链式调用
- 图解
![](https://user-gold-cdn.xitu.io/2020/1/14/16fa329e240b2fd7?w=1440&h=1920&f=png&s=2932187)
- 示例源码，比如这个 chain.proceed(requestBuilder.build()); 语句前就是前置，语句后就是后置。proceed方法表示调用下一个interceptor，前置最全走完就开始走后置，大约是个轮回。
```java
    // 前置工作
    if (userRequest.header("User-Agent") == null) {
      requestBuilder.header("User-Agent", Version.userAgent());
    }

    Response networkResponse = chain.proceed(requestBuilder.build());
    
    // 后置
    HttpHeaders.receiveHeaders(cookieJar, userRequest.url(), networkResponse.headers());
    Response.Builder responseBuilder = networkResponse.newBuilder()
        .request(userRequest);
  
```
## 各个interceptor的作用，按顺序来的
- 如果你设置了interceptor，那就是在这执行
- retryAndFollowUpInterceptor。前置：无前置，如果你不自定义的interceptor的话，他就是第一个，第一个要毛线前置工作。后置：看是否需要转发或者重定向，如果是的话，又开始一次轮回的interceptors链调用。
- BridgeInterceptor。前置：添加各种HEAD，比如Content-Type等等，还有okHttp给我们实现了gzip解码，所以如果你不指定压缩格式的话，会默认加一个Accept-Encoding=gzip的头。后置：如果需要解压gzip的话，就解压。还有Cookie也是在这做的。
- CacheInterceptor。前置：如果有Cache的话，就直接创建一个Response返回去，就不走后面的interceptor了，接着就走BridgeInterceptor的后置xxxxxx，差不多就这个意思。后置：存cache。
- ConnectInterceptor
- CallServerInterceptor 这两个就是真正发请求的地方了，使用OkIo的那一套。牛逼就完事儿了。

## 理解OkHttp各个变量
- Dispatcher dispatcher： 做线程调度的。
- Proxy proxy：做代理用的，翻墙用过吗？大兄弟。
- List<Protocol> protocols：支持的协议，比如http/1.0 / http/1.1 / http/2.0 之类的。没啥用。
- List<ConnectionSpec> connectionSpecs。如果是Https连接的话，这里表示支持的非对称加密算法，对称加密的算法等等。没啥用+1。
- List<Interceptor> interceptors：贼鸡儿有用。如果interceptors链从这里调用。
- List<Interceptor> networkInterceptors。没啥用，这个调用时机在CallServerInterceptor之前，就是倒数第二个，拿到的都是原始数据，二进制。我觉得没啥用就没在上面的interceptor中写出。
- EventListener.Factory eventListenerFactory： 没啥用+1
- ProxySelector proxySelector： 没啥用+1
- CookieJar cookieJar：有点用，不是很有用。cookie存取需要自己实现。
- Cache cache：配置Cache参数的，大小等。没啥用。
- Authenticator authenticator： 这个如果配置了，如果遇到了403权限不足错误的话，就会回调这个方法，当然了这个方法是你自己写的。
- ConnectionPool connectionPool: 连接池，不动他就可以了，有用，但是我不用！
- 剩下的 没啥好说的了，我列一下。
```
Dns dns;
boolean followSslRedirects;
boolean followRedirects;
boolean retryOnConnectionFailure;
int connectTimeout;
int readTimeout;
int writeTimeout;
int pingInterval;
```

# Retrofit分析
## 核心是动态代理。
- retrofit对象调用create方法后（传入一个interface），然后retrofit会通过Proxy.newProxyInstance方法创造一个对象去实现这个interface，这个interface里面每个方法实现方式基本差不多，都是调用InvocationHandler对象的invoke方法。

## invoke干了啥
### 第一行： ServiceMethod<Object, Object> serviceMethod = (ServiceMethod<Object, Object>) loadServiceMethod(method);
- 解析interface里每个方法的信息，比如返回值类型，方法注解，参数类型等，并做分析

### 第二行： OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
- 创建了一个Retrofit的OkHttpCall，注意是Retrofit的，不是okhttp包下面的，然后把第一行得到的serviceMethod封装到这个对象里面，到时候就可以用serviceMethod里面的信息创造一个okhttp的call对象，然后就可以用这个okhttp的call对象进行真正的网络请求了。

### 第三行： return serviceMethod.adapt(okHttpCall);
- 这个方法会使用ServiceMethod对象中的CallAdapter对象来把okHttpCall（Retrofit的okHttpCall)进行转换，生成一个新的retrofit2.call对象中，然后在这个新年的对象中，进行线程的切换以及真正的网络请求。另外Rxjava的适配也是在这儿。

## 大概流程
- 调用service(及interface)后，再调用enqueue / execute  方法。里面实际调用的是okhttp的execute方法。然后再进行解析response，比如非2xx会直接抛出错误，或者你Call填的泛形是这个具体对象的话，这里也可以帮你直接json转对象。