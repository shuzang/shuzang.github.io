# Golang设置与使用cookie


关于 Cookie 的使用是 Web 编程中的重要一部分，本篇介绍 Cookie 的基本知识和 Golang 中使用的方法。

<!--more-->

## 1. 使用Cookie来管理状态

HTTP 是无状态协议，不记录之前发生过的请求和响应，也因此无法根据历史状态信息处理当前请求。但假设我们正在浏览淘宝，然后在首页进行了登录，点击并跳转到商品页面时，因为 HTTP 的无状态特性，就需要重新进行登录，这带来了诸多不便。

不可否认，无状态协议有它的优点，由于不必保存状态，减少了服务器的 CPU 及内存资源消耗，同时也因为协议足够简单，才可以用到各种场景中。那么如何解决逛淘宝这种需要记录历史信息的场景呢，这里就用到 Cookie 技术。

Cookie 技术通过在 HTTP 请求和响应报文中写入 Cookie 信息来控制客户端的状态。首先，服务端在发送的响应报文内添加一个叫做 Set-Cookie 的首部字段信息，客户端接收到后会保存 Cookie。当下次客户端向服务器发送请求时，就会自动在请求报文中加入保存的 Cookie 值。服务器发现客户端发送过来的 Cookie 后，会检查究竟是哪个客户端发来的连接请求，然后对比服务器上的数据，得到之前的状态信息。过程如下图所示（图源为《图解HTTP》）

![第一次请求时服务端在响应报文中添加Cookie](/images/Golang设置与使用cookie/epub_907764_48.jpg)



![之后自动在请求报文中添加Cookie](/images/Golang设置与使用cookie/epub_907764_50.jpg)

所以 Cookie 的实质是 HTTP 请求与响应报文中的一个首部字段信息，下面给出一些报文示例：

1. 请求报文（没有 Cookie 信息时的状态）

   ```
   GET /reader/ HTTP/1.1
   Host: hackr.jp
   *首部字段内没有Cookie的相关信息
   ```

2. 响应报文（服务器端生成 Cookie 信息）

   ```
   HTTP/1.1 200 OK
   Date: Thu, 12 Jul 2012 07:12:20 GMT
   Server: Apache
   ＜Set-Cookie: sid=1342077140226724; path=/; expires=Wed,10-Oct-12 07:12:20 GMT＞
   Content-Type: text/plain; charset=UTF-8
   ```

3. 请求报文（自动发送保存的 Cookie 信息）

   ```
   GET /image/ HTTP/1.1
   Host: hackr.jp
   Cookie: sid=1342077140226724
   ```

## 2. 关于Cookie的首部字段

关于 Cookie 的相关说明没有被编入标准化 HTTP/1.1 的RFC2516中，下面介绍的是使用最广泛的 Cookie 标准。

| 首部字段名 | 说明                           | 首部类型     |
| ---------- | ------------------------------ | ------------ |
| Set-Cookie | 开始状态管理所使用的Cookie信息 | 响应首部字段 |
| Cookie     | 服务器接收到的Cookie信息       | 请求首部字段 |

### 2.1 Set-Cookie

```
Set-Cookie: status=enable; expires=Tue, 05 Jul 2011 07:26:31 GMT; ⇒
path=/; domain=.hackr.jp;
```

当服务器准备开始管理客户端的状态时，会事先告知各种信息。下面的表格列举了Set-Cookie 的字段值。

| 属性         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| NAME=VALUE   | 赋予Cookie的名称和其值（必须项）                             |
| expires=DATE | Cookie的有效期（若不明确指定则默认为浏览器关闭前为止）       |
| path=PATH    | 将服务器上的文件目录作为Cookie的适用对象（若不指定则默认为文档所在的文件目录） |
| domain=域名  | 作为Cookie适用对象的域名（若不指定则默认为创建Cookie的服务器域名） |
| Secure       | 仅在HTTPS安全通信时才会发送Cookie                            |
| HttpOnly     | 加以限制，使Cookie不能被JavaScript脚本访问                   |

**domain属性**：domain 属性指定的域名可做到与结尾匹配一致，比如，当指定 example.com 后，除 example.com 本身外，www.example.com 或 www2.example.com 等都可以发送 Cookie，所以，除了针对具体指定的多个域名发送Cookie之外，不指定domain属性显得更安全

**secure属性**：指定 secure 属性的方法为 `Set-Cookie: name=value; secure`

**HttpOnly属性**：指定 HttpOnly 属性的方法为 `Set-Cookie: name=value; HttpOnly`

### 2.2 Cookie

```
Cookie: status=enable
```

在请求报文中添加该字段后，就相当于告诉服务器客户端想要获得 HTTP 状态管理支持。接收到多个Cookie时，同样可以以多个Cookie形式发送。

## 3. Session 管理



## 参考

1. 上野宣[日]，[图解HTTP](https://book.douban.com/subject/25863515/)，人民邮电出版社，2014
2. 学院君，[在 Go 语言中设置、读取和删除 HTTP Cookie](https://xueyuanjun.com/post/21668)，2020
