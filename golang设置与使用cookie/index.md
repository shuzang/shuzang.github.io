# Golang设置与使用cookie


关于 Cookie 的使用是 Web 编程中的重要一部分，本篇介绍 Cookie 的基本知识和 Golang 中使用的方法。

<!--more-->

## 1. 使用Cookie来管理状态

HTTP 是无状态协议，不记录之前发生过的请求和响应，也因此无法根据历史状态信息处理当前请求。但假设我们正在浏览淘宝，然后在首页进行了登录，点击并跳转到商品页面时，因为 HTTP 的无状态特性，就需要重新进行登录，这带来了诸多不便。

不可否认，无状态协议有它的优点，由于不必保存状态，减少了服务器的 CPU 及内存资源消耗，同时也因为协议足够简单，才可以用到各种场景中。那么如何解决逛淘宝这种需要记录历史信息的场景呢，这里就用到 Cookie 技术。

Cookie 技术通过在 HTTP 请求和响应报文中写入 Cookie 信息来控制客户端的状态。首先，服务端在发送的响应报文内添加一个叫做 Set-Cookie 的首部字段信息，客户端接收到后会保存 Cookie。当下次客户端向服务器发送请求时，就会自动在请求报文中加入保存的 Cookie 值。服务器发现客户端发送过来的 Cookie 后，会检查究竟是哪个客户端发来的连接请求，然后对比服务器上的数据，得到之前的状态信息。过程如下图所示（图源为《图解HTTP》）

![第一次请求时服务端在响应报文中添加Cookie](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20200601_epub_907764_48.jpg)



![之后自动在请求报文中添加Cookie](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20200601_epub_907764_50.jpg)

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

**expires属性**：有两种方法来设置过期时间：一种是直接设置 `Expires` 字段，一种是设置 `MaxAge` 字段。前者表示到期的具体时间点，后者表示 Cookie 的有效时长（单位是秒）。这并不是 Go 语言的设计，而是不同浏览器的混乱标准使然，比如虽然 HTTP/1.1 有意废弃 `Expires`，不过 IE 6、7、8 却不支持 `MaxAge` 字段。通常，考虑到默认时区问题，本地时间不可靠，推荐通过 `MaxAge` 字段设置 Cookie 过期时间，不过对于 Web 应用而言，通常不设置过期时间，让 Cookie 随着浏览器关闭而失效即可。

**domain属性**：domain 属性指定的域名可做到与结尾匹配一致，比如，当指定 example.com 后，除 example.com 本身外，www.example.com 或 www2.example.com 等都可以发送 Cookie，所以，除了针对具体指定的多个域名发送Cookie之外，不指定domain属性显得更安全

**secure属性**：指定 secure 属性的方法为 `Set-Cookie: name=value; secure`

**HttpOnly属性**：指定 HttpOnly 属性的方法为 `Set-Cookie: name=value; HttpOnly`

### 2.2 Cookie

```
Cookie: status=enable
```

在请求报文中添加该字段后，就相当于告诉服务器客户端想要获得 HTTP 状态管理支持。接收到多个Cookie时，同样可以以多个Cookie形式发送。

## 3. Session 管理

某些 Web 页面只想让特定的人浏览，或者干脆仅本人可见，为达到这个目标，需要添加认证功能。HTTP/1.1 实用的认证包括 BASIC认证、DIGEST认证、SSL客户端认证、FormBase认证等，由于使用上的便利性和安全性问题，前两种几乎不适用，SSL客户端认证则由于导入及费用问题未得到普及，目前常用的是最后一种：基于表单的认证。

基于表单的认证方法并不是在HTTP协议中定义的，而是由客户端通过表单向服务器提交登录信息，然后由服务器安装自定义的实现方式进行验证，不同的应用使用的验证方式多有不同，但多数情况下，是基于用户输入的用户ID（通常是任意字符串或邮件地址）和密码等登录信息进行认证。

鉴于 HTTP 是无状态协议，之前已认证成功额用户状态无法保留，因此一般使用 Cookie 来管理 Session(会话)。

![](https://picped-1301226557.cos.ap-beijing.myqcloud.com/Go_20200601_epub_907764_193.jpg)

如上图所示，基本的步骤为

1. 客户端把用户ID和密码等登录信息放入报文的实体部分，通常是以POST方法把请求发送给服务器。而这时，会使用HTTPS通信来进行HTML表单画面的显示和用户输入数据的发送。
2. 服务器会发放用以识别用户的Session ID。通过验证从客户端发送过来的登录信息进行身份认证，然后把用户的认证状态与Session ID绑定后记录在服务器端。向客户端返回响应时，会在首部字段Set-Cookie内写入Session ID（如PHPSESSID=028a8c…）。你可以把Session ID想象成一种用以区分不同用户的等位号。然而，如果Session ID被第三方盗走，对方就可以伪装成你的身份进行恶意操作了。因此必须防止Session ID被盗，或被猜出。为了做到这点，Session ID应使用难以推测的字符串，且服务器端也需要进行有效期的管理，保证其安全性。另外，为减轻跨站脚本攻击（XSS）造成的损失，建议事先在Cookie内加上httponly属性。
3. 客户端接收到从服务器端发来的Session ID后，会将其作为Cookie保存在本地。下次向服务器发送请求时，浏览器会自动发送Cookie，所以SessionID也随之发送到服务器。服务器端可通过验证接收到的Session ID识别用户和其认证状态。

需要注意，上述介绍并不是唯一的实现方式，实际上，不仅基于表单认证的登录信息及认证过程没有标准化，服务端如何保持密码等登录信息也没有标准化。通常，一种安全的保存方法是，先利用给密码加盐（salt）的方式增加额外信息，再使用散列（hash）函数计算出散列值后保存。但是我们也经常看到直接保存明文密码的做法，而这样的做法具有导致密码泄露的风险。

## 4. Go中Cookie的设置与读取

### 4.1 设置Cookie

Go语言中通过 net/http 包中的 SetCookie 来设置 Cookie：

```go
http.SetCookie(w ResponseWriter, cookie *Cookie)
```

w 表示需要写入的 response，cookie 是一个 struct，让我们来看看对象是怎样的：

```go
type Cookie str、uct {
    Name        string
    Value       string
    Path        string
    Domain      string
    Expires     time.Time
    RawExpires  string
    // MaxAge=0 意味着没有指定 Max-Age 的值
    // MaxAge<0 意味着现在就删除 Cookie，等价于 Max-Age=0
    // MaxAge>0 意味着 Max-Age 属性存在并以秒为单位存在
    MaxAge      int
    Secure      bool
    HttpOnly    bool
    Raw         string
    Unparsed    []string // 未解析的 attribute-value 属性位对
}
```

下面来看一个如何设置 Cookie 的例子：

```go
expiration := time.Now()
expiration := expiration.AddDate(1, 0, 0)
cookie := http.Cookie{
    Name: "username", 
    Value: "zuolan", 
    Expires: expiration
}
http.SetCookie(writer, &Cookie)
```

### 4.2 读取 Cookie

上面的例子演示了如何设置 Cookie 数据，这里演示如何读取 Cookie：

```go
cookie, _ := r.Cookie("username")
fmt.Fprint(w, cookie)
```

还有另外一种读取方式：

```go
for _, cookie := range r.Cookies() {    
    fmt.Fprint(w, cookie.Name)
}
```

可以看到通过 request 获取 Cookie 非常方便。

## 参考

1. 上野宣[日]，[图解HTTP](https://book.douban.com/subject/25863515/)，人民邮电出版社，2014
2. 学院君，[在 Go 语言中设置、读取和删除 HTTP Cookie](https://xueyuanjun.com/post/21668)，2020
