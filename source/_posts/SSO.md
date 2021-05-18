---
title: web前端的单点登录-三种实现方案
date: 2021-04-29 19:17:09
tags: [前端]
categories: [前端]
---

https://zcy-cdn.oss-cn-shanghai.aliyuncs.com/f2e-assets/ebf178b6-0950-482b-8031-5bb70b4888d5.png?x-oss-process=image/quality,Q_75/format,jpg

## 背景

公司的统一登陆平台，也就是单点登录系统，因为某些地方不合理，需要重构，当我着手开始重构项目的时候意外的发现，单点登陆大概有三种实现的方案，所以详细的调研学习了一下三种实现的方案，并简单的对比了一下。

## 什么是单点登录

单点登陆，又叫做SSO（Single Sign On）

我从网上抄了一段话来解释一下点单登陆的作用：是目前比较流行的企业业务整合的解决方案之一。SSO的定义是在多个应用系统中，用户只需要登录一次就可以访问所有相互信任的应用系统。

例如百度公司下的百度地图和百度贴吧，用户只要登录一次，就能访问百度公司下的所有应用

所以简单来说SSO的目的就是多个相互信任的系统之间共享登陆的状态

## 实现原理

### 单个系统实现登陆

实现单个系统的登陆的时候，我们通常使用浏览器的cookie来存储用户的登陆信息，

首先会正常的请求接口，然后判断是否登陆，如果没有登陆则直接跳转登录的页面，用户输入用户名和密码登录，后端验证过登陆信息后，返回登录成功的状态，并且返回一个用户登录状态的token，把token存在浏览器的cookie里面，下次访问接口的时候把这个token带上，只要token在有效期之内，服务端就会通过验证，不需要再次登录

### 多个系统实现单点登陆

单点登录的本质就是要在多个系统中共享登录状态，也就是token，那么如何让多个系统共享一个登录状态呢？

## 实现单点登录的三种方法

我是前端，所以这里只讨论前端的实现

### cookie提升

简单了解一下cookie的机制：

1. cookie是存储在用户浏览器本地的一种方案，可以由服务器响应报文Set-Cookie的首部字段信息或者客户端 document.cookie来设置，并随着每次请求发送到服务器。

2. cookie是有域的限制的，不同的域名之间的cookie是不能共享的，但是子域名可以获取父级域名 Cookie。这是关cookie实现单点登录的关键

cookie的设置：

```javascript
document.cookie = "cookieName=mader; expires=Fri, 31 Dec 2017 15:59:59 GMT; path=/mydir; domain=cnblogs.com; max-age=3600; secure=true";
```

- cookieName=mader ：name=value，cookie的名称和值

- expires=Fri, 31 Dec 2017 15:59:59 GMT： expires，cookie过期的日期，如果没有定义，cookie会在对话结束时过期。日期格式为 new Date().toUTCString()

- path=/mydir: path=path (例如 '/', '/mydir') 如果没有定义，默认为当前文档位置的路径。

- domain=xxx.com： 指定域, 如果没有定义，默认为当前文档位置的路径的域名部分。

- max-age=3600： 文档被查看后cookie过期时间，单位为秒

- secure=true： cookie只会被https传输 ，即加密的https链接传输

浏览器会将domain和path都相同的cookie保存在一个文件里，cookie间用*隔开

一般我们公司要实现单点登录只需要做自己公司的不同系统之间的登录状态共享，一般会有一个主域名，然后在主域名下申请二级域名给不同的系统部署，例如:百度的主域名是’baidu.com‘，而百度贴吧和百度地图的域名就是在主域名下面的二级域名：‘tieba.baidu.com’和‘map.baidu.com’

利用cookie子域可以获取父域的特点，可是设置cookie的domain属性为最外的一级域名，path属性设置为‘/’，子域就可以获取父域的cookie了

所有的域名都能共享一个token，传给服务端的时候服务端就会判断我们那已经登录过了

### CAS

CAS（Central Authentication Service），即中央认证服务，也可以叫做认证中心，是 Yale 大学发起的一个开源项目，目的是为 Web 应用系统提供一种可靠的单点登录方法。

中央认证，就是专门负责登录请求的web服务。认证中心是单点登陆的标准做法，支持跨域且扩展性好，但是实现起来稍微麻烦

假设我们有两个相互信任的系统：a.xx.com 和 b.xx.com，这两个系统要实现单点登录，所以需要一个独立部署的中央认证登陆系统，例如：login.xx.com。

因为中央认证服务是需要单独部署的一个应用，所以除了我们当前开发的项目的服务器，还另外需要一个中央认证的服务器，也就是CAS服务器

首先是用户第一次进入a系统，需要登录：

图片站位

1. 用户在浏览器输入a系统的网址：a.xx.com，访问a系统对应的a服务器

2. a服务器验证用户并没有登录，重定向到CAS服务器的登录页面，并且在重定向的地址后面携带登录成功后要返回的a系统的页面url，通常像这样子：https://ex-cas.com/login?service=https://a.xx.com

3. 用户访问CAS的登录系统，CAS判断用户是否有CAS的session，也就是判断浏览器是否存有CAS的session

4. CAS判断用户并没有CAS的session，返回CAS的登录页面

5. 用户输入：用户名和密码点击提交，提交到CAS服务器

6. CAS服务器验证用户信息，创建CAS的session，通过set-cookie写入用户的浏览器，

7. 然后重定向到a.xx.com，并且在重定向的时候CAS服务器会在a.xx.com后面拼上CAS的一个ticket字段，例如：https://a.xx.com?ticket=123456

8. 这个就可以正常的访问a系统的应用了，然后a.xx.com会访问a系统对应的a服务器，a服务器会获取url后面的ticket，

9. a服务器拿ticket去向CAS服务器验证，验证成功之后，CAS服务器会返回验证成功的消息和一些必要的信息

10. a服务器通过set-cookie设置自己的session，然后重定向到https://a.xx.com，去掉后面的ticket

11. 然后用户访问https://a.xx.com，a服务器验证用户是否登录，返回请求内容

整个过程有三次重定向，两次cookie的写入，还是比较耗费资源的

然后用户第二次进入a系统，假设第一次登录的信息还没过期，不需要登录：

图片站位

1. 用户第二次在浏览器输入https://a.xx.com，访问a系统对应的a服务器

2. a服务器验证用户登录信息，用户有a服务器的session，判断用户是登陆状态

3. 返回请求内容

用户第一次进入b系统，因为在进入a系统的时候已经在CAS中央认证服务登录过了，所以也不需要登陆：

图片站位

1. 用户第一次在浏览器输入b系统的网址:b.xx.com，访问b系统对应的b服务器

2. b服务器验证用户并没有登录，重定向到CAS服务器的登录页面，并且在重定向的地址后面携带登录成功后要返回的b系统的页面url，通常像这样子：https://ex-cas.com/login?service=https://b.xx.com

3. 用户访问https://ex-cas.com/login?service=https://b.xx.com，通过cookie携带CAS服务器的session

4. CAS服务器验证CAS的session，验证用户已经登陆过了，不需要重新登陆，直接携带ticket，重定向到https://b.xx.com?ticket=123456

5. 用户访问https://b.xx.com?ticket=123456，b服务器拿ticket去向CAS服务器验证，验证成功之后，CAS服务器会返回验证成功的消息和一些必要的信息

6. b服务器通过set-cookie设置自己的session，然后重定向到https://b.xx.com，去掉后面的ticket

7. 然后用户访问https://a.xx.com，a服务器验证用户是否登录，返回请求内容

### LocalStorage 跨域

这种方案呢，用的人比较少，我也只是简单的了解了一下原理

假设我们有两个相互信任的系统a.xx.com和b.xx.com要实现单点登录，大概的思路：

在a.xx.com/login的页面里面通过内嵌一个iframe页面，在这个iframe里面通过storage把用户的登陆session放入iframe父级下面存储起来。

通过这种方式实现登录状态的共享
