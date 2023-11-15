---
date: 2023-11-15 20:12:02
linktitle: cooike、session和sso

title: cooike、session和sso
weight: 10
tags : [
    "cooike、session和sso",
]
---

## 1. 引子

### http的无状态

- http的无状态，第一次访问和第二次访问是完全独立的，需要将不同的请求的行为串起来，那就让服务器和浏览器共同维护一个状态吧！这就是会话机制。（浏览器第一次请求服务器，服务器创建一个会话，并将会话的id作为响应的一部分发送给浏览器，浏览器存储会话id，并在后续第二次和第三次请求中带上会话id，服务器取得请求中的会话id就知道是不是同一个用户了，这个过程用下图说明，后续请求与第一次请求产生了关联）

```
服务器在内存中保存会话对象，浏览器怎么保存会话id呢？你可能会想到两种方式
请求参数
cookie
将会话id作为每一个请求的参数，服务器接收请求自然能解析参数获得会话id，并借此判断是否来自同一会话，很明显，这种方式不靠谱。那就浏览器自己来维护这个会话id吧，每次发送http请求时浏览器自动发送会话id，cookie机制正好用来做这件事。cookie是浏览器用来存储少量数据的一种机制，数据以”key/value“形式存储，浏览器发送http请求时自动附带cookie信息
tomcat会话机制当然也实现了cookie，访问tomcat服务器时，浏览器中可以看到一个名为“JSESSIONID”的cookie，这就是tomcat会话机制维护的会话id，使用了cookie的请求响应过程如下图：

单系统登录解决方案的核心是cookie，cookie携带会话id（sessionId）在浏览器与服务器之间维护会话状态。但cookie是有限制的，这个限制就是cookie的域（通常对应网站的域名），浏览器发送http请求时会自动携带与该域匹配的cookie，而不是所有cookie

既然这样，为什么不将web应用群中所有子系统的域名统一在一个顶级域名下，例如“*.baidu.com”，然后将它们的cookie域设置为“baidu.com”，这种做法理论上是可以的，甚至早期很多多系统登录就采用这种同域名共享cookie的方式。
然而，可行并不代表好，共享cookie的方式存在众多局限。首先，应用群域名得统一；其次，应用群各系统使用的技术（至少是web服务器）要相同，不然cookie的key值（tomcat为JSESSIONID）不同，无法维持会话，共享cookie的方式是无法实现跨语言技术平台登录的，比如java、php、.net系统之间；第三，cookie本身不安全。

共享 Cookie 的应用系统必须协商并统一使用相同的 Cookie 名称来标识用户会话。不能tomcat教JSESSIONID。php又叫xxxxx。或者用更安全和可行的方法是使用标准的身份验证和授权协议，如OAuth或OpenID Connect。这些协议定义了一致的方式来处理用户身份验证和会话管理，而不依赖于特定的 Cookie 名称。

```

------



```
传统验证用户信息用的是session和cookie这种组合，session存在服务器上，cookie存在客户端。cookie由服务器生成的，发给客户端存着，客户端每次请求就会携带cookie，服务器就会通过cookie的内容从而识别客户端的身份。session存在服务器，客户端的请求都会携带自己的身份信息，服务器就会通过session对比这个信息判断请求是否合法。但是session用户过多开销过大，也不能在多个服务器的服务端下使用，因为它存在服务器的内存中无法共享。
```



## 2. **会话机制**

1.  过程：

      1 服务端生成sessionId存储并返回

      2 客户端的each 请求默认传cookie(里面塞了sessionId)

    缺点：

      1 cookie无法解决 跨域、跨语言平台的问题（cookie有限制，不能跨域，只能发送域下匹配的所有cookie），因为cookie是跟浏览器以及各自的语言框架耦合集成的

   ​    2 服务端需要存储，传统的session存在服务器内存中、redis中，浪费内存https://zhuanlan.zhihu.com/p/135247813

   


```
前面已经回答的挺透彻的了，补充一点概念辨析权当拾遗。

SSO本质上属于一种需求。

CAS（中央认证服务）本质上是一个框架，用于实现SSO的需求，但是并不提供具体的解决方案，只是规定了CAS Server和CAS Client的关系，其中中央认证服务器必须单点部署，而其他可以分布部署。

OAuth2是一个协议，用于解决CAS框架下，客户资源安全的问题。通过中央认证服务器提供的Token来保证用户资源不会被窃取。同样SAML也是一个协议。

JWT是一种具体的技术实现方案，通过在请求头部置入一个加密后的多段式Json（一般至少是三段：header, payload, signature）来保证token的可靠性。
```



## 3.单点登录（SSO）

 简述： N个系统   只要其中1个登录，其他系统不需要重复认证登录了

 单点登录四大主流协议： CAS、OAuth、OpenID Connect、SAML方案

 [单点登录](https://www.zhihu.com/search?q=单点登录&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A2981474606})有很多实现方案，最主流的还是基于CAS的方案。设计一套支持单点登录的[鉴权认证](https://www.zhihu.com/search?q=鉴权认证&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"answer"%2C"sourceId"%3A2981474606})系统，所有系统都基于一套鉴权系统进行登录，并且可以实现各个系统之间的互信和跳转。CAS架构的核心是需要搭建一个CAS Server，该服务独立部署，拥有独立三级域名，主要负责对用户的认证工作。他主要组成包括WEB前端提供登录页面，票据模块，认证模块。



```

在 B/S 系统中，登录功能通常都是基于 Cookie 来实现的。当用户登录成功后，一般会将登录状态记录到 Session 中，或者是给用户签发一个 Token，无论哪一种方式，都需要在客户端保存一些信息（Session ID 或 Token ），并要求客户端在之后的每次请求中携带它们。在这样的场景下，使用 Cookie 无疑是最方便的，因此我们一般都会将 Session 的 ID 或 Token 保存到 Cookie 中，当服务端收到请求后，通过验证 Cookie 中的信息来判断用户是否登录 。
单点登录（Single Sign On, SSO）是指在同一帐号平台下的多个应用系统中，用户只需登录一次，即可访问所有相互信任的应用系统。举例来说，百度贴吧和百度地图是百度公司旗下的两个不同的应用系统，如果用户在百度贴吧登录过之后，当他访问百度地图时无需再次登录，那么就说明百度贴吧和百度地图之间实现了单点登录。
单点登录的本质就是在多个应用系统中共享登录状态。如果用户的登录状态是记录在 Session 中的，要实现共享登录状态，就要先共享 Session，比如可以将 Session 序列化到 Redis 中，让多个应用系统共享同一个 Redis，直接读取 Redis 来获取 Session。当然仅此是不够的，因为不同的应用系统有着不同的域名，尽管 Session 共享了，但是由于 Session ID 是往往保存在浏览器 Cookie 中的，因此存在作用域的限制，无法跨域名传递，也就是说当用户在 http://app1.com 中登录后，Session ID 仅在浏览器访问 http://app1.com 时才会自动在请求头中携带，而当浏览器访问 http://app2.com 时，Session ID 是不会被带过去的。实现单点登录的关键在于，如何让 Session ID（或 Token）在多个域中共享。



```



------

```
方案 : OAuth2 + jwt token
实现 : Spring Security + Spring OAuth2
这个方案最好是前后端完全分离的架构
另外这个方案唯一个比较麻烦的问题是权限revoke的问题, 毕竟不需要保留token状态, 但可以把token有效期弄短点(反正有refresh token) + 或者某种有效期内的黑名单机制, 以及其它一些办法解决. 
一般授权服务都是集中式倒是好说
对于资源服务, 如果你是分布式部署(比如k8s), 但通过统一的api入口来实现外部api访问的权限控制, 比如Kong, ingress, 或者nginx, 或者ServiceMesh之类, 那你可以把黑名单机制放一个地方实现
如果式分散式控制, 比如spring resource server,  就只有走某种share存储机制来保存黑名单了, 比如redis


作者：步行鸟
链接：https://www.zhihu.com/question/342103776/answer/801395280
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



```

### SSO常见问题

1  是否支持多域名

2 手机端很多浏览器不支持cookie或者是禁用cookie



## 4.JWT

### 4.1 介绍

​	按照某种加密规则对 业务信息(如UserId)进行加密得到1个token（ 三个由点分隔的Base64-URL字符串），用token来代替XXX，进行后续的**登陆鉴权**等。

​      **本质上是对 通过加密算法对业务信息的加密映射  ，来增加服务端压力和业务安全性**

### 4.2 优缺点

 **优点**：   1 加密安全

  			 2  开销很小，token不用存，只需要解密，减少查库  

​             3  能够轻松地跨不同域使用

 **缺点：**

  安全性：jwt本身是不可伪造，不可篡改的，但是不代表非法用户冒充正常用法发起请求，所以常规的几个安全策略在实际项目中都应该使用：

   1 使用https  

   2 使用http-only的cookie，针对sid和jwt  -----只让服务器修改cookie的

   3 管理好密钥

   4 防范CSRF攻击

   过期时间不要太长，1个小时？（  关于过期时间太长  但是jwt在我看来也是有问题了，签发出去服务器就不管了，我利用这两个小时的过期时间可以做很多事情，可以用频率检测，ip检测来限制这种我这种用户 )

### 4.3 使用细节

​      **整体流程**：

​       用户发起登陆===》服务端check密码===》 通过 generate token 返回 ，之后每次 前端带个token，

 后端filter 检验token是否过期、不过期则放行

登录成功：

  当用户成功登录时，JWT将被返回并且必须保存在本地（通常在本地存储中，但也可以使用Cookie），而不是在传统方法中创建会话 服务器并返回一个cookie。

token 在客户端一般存放于localStorage、cookie、或sessionStorage

​       1 localStorage可存储持久化的数据;

​       2  sessionStorage仅限于当前窗口

​       3 cookie不能跨域（ 可能有csrf 跨站请求伪造)

​           我们方案中token是存在localStorage

#### **鉴权**

一旦用户登录，每个后续请求都将包含JWT，允许用户访问该令牌允许的路由，服务和资源。

密钥实践

​          按我们的实践，签名都是加盐然后再加长密钥加密

#### token过期续签  

2种方案

 1 重定向登陆页面，登陆成功后刷新token后 调到之前页面

​        2  后台增加 刷新token的接口

recloud实践

1登陆成功  拿到2个token

   第一个是 acessToken ，它的过期时间比较短，不如1天；另外一个是 refreshToken 它的过期时间更长一点比如为10天。客户端登录后，将 accessToken和refreshToken 保存在客户端本地，每次访问将 accessToken 传给服务端。服务端校验 accessToken 的有效性，如果过期的话，就将 refreshToken 传给服务端。如果 refreshToken 有效，服务端就生成新的 accessToken 给客户端。否则，客户端就重新登录即可。

​    2 什么时候会让用户重新登陆

localstorage的username为空的时候，主动退出会清掉这个

https://developer.aliyun.com/article/1329396

https://www.jb51.net/article/264915.html

```

jwt
1 组成：jwt组成 2个点隔开的3部分
   Header.Payload.Signature           如xxxxx.yyyyy.zzzzz
2 Header和Payload 是base64编码的，非加密， Signature是用密钥加密的
3 生成方式就是前两部分分别base64加密组成字符串再加上密钥这个字符串再通过第一部分声明的HS256方式加密得出来了的。HS256就是一种对称加解密。加解密就是一个密钥。



详解解释：
1 Header通常由两部分组成：令牌的类型，即JWT。和常用的散列算法，如HMAC SHA256或RSA。
{
  "alg": "HS256",
  "typ": "JWT"
}


2 声明内容
Registered claims（注册声明）:
    这些是一组预先定义的声明，它们不是强制性的，但推荐使用，以提供一组有用的，可互操作的声明。 其中一些是：iss（发行者），exp（到期时间），sub（主题），aud（受众）等。

Public claims（公开声明）:可以由使用JWT的人员随意定义
Private claims（私有声明）:为了同意使用它们但是既没有登记，也没有公开声明的各方之间共享信息，而创建的定制声明。

示例：
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}

3 第三部分signature用来验证发送请求者身份，由前两部分加密形成。


eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiIsImtpZCI6InRva2VuIn0.eyJzdWIiOiJhYzcwOThiZi05OWQ5LTM1NDEtODI3NC05Y2ZjYjRjNTZlYTQiLCJhdWQiOiJyZWNsb3VkIiwiZXhwIjoxNjk5MDcyNDg3fQ.Nvqzqv5TL4o1z1jukZsil6CjrpG7SGAsEk-8vJT_tmHBK_pDgPC5q-1zqQ3Y40eMR1OidroCeeIlaZ2NOv7kLw

eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiIsImtpZCI6InRva2VuIn0=
base64解码后
{
  "typ": "JWT",
  "alg": "HS512",
  "kid": "token"
}


eyJzdWIiOiJhYzcwOThiZi05OWQ5LTM1NDEtODI3NC05Y2ZjYjRjNTZlYTQiLCJhdWQiOiJyZWNsb3VkIiwiZXhwIjoxNjk5MDcyNDg3fQ==
base64解码后
{
  "sub": "ac7098bf-99d9-3541-8274-9cfcb4c56ea4",
  "aud": "recloud",
  "exp": 1699072487
}


```





用户登录状态的存储与校验

通常实体类

User {

   token：XXX，

   user_id:  1

}





  浏览器把服务端返回的session存到cookie中（下次发送http请求时会自动携带与该域匹配的cookie）   服务器生成session并存在内存中
