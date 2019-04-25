---
title: 聊聊cookie(转)
date: 2019-04-25 14:04:47
tags: cookie
---

[参考链接1](https://juejin.im/post/59d1f59bf265da06700b0934)
[参考链接2](https://segmentfault.com/a/1190000004556040)

## cookie是什么

这个讲起来很简单，了解http的同学，肯定知道，http是一个不保存状态的协议，什么叫不保存状态，就是一个服务器是不清楚是不是同一个浏览器在访问他，在cookie之前，有另外的技术是可以解决，这里简单讲一下，就是在请求中插入一个token，然后在发送请求的时候，把这个东西带给服务器，这种方式是易出错，所以有了cookie的出现

![cookie是什么](https://user-gold-cdn.xitu.io/2017/10/2/9749a2f293a5b4f84d8a40b4e8657faf?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## `cookie`原理

![cookie原理](https://user-gold-cdn.xitu.io/2017/10/2/07ecb36c4820a66de90013f303cac8c0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

第一次访问网站的时候，浏览器发出请求，服务器响应请求后，会将cookie放入到响应请求中，在浏览器第二次发请求的时候，会把cookie带过去，服务端会辨别用户身份，当然服务器也可以修改cookie内容

## `cookie`不可跨域

我就几个例子你就懂了，当我打开百度的网页，我要设置一个cookie的时候，我的指令如下：

```javascript
document.cookie='myname=laihuamin;path=/;domain=.baidu.com';
document.cookie='myname=huaminlai;path=/;domain=.google.com';
```

当我将这两个语句都放到浏览器控制台运行的时候，你会发现一点,注意，上面两个cookie的值是不相同的，看清楚:

![cookie不可跨域](https://user-gold-cdn.xitu.io/2017/10/2/ef12b6b1b2590434c959161d39fc7adc?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

显而易见的是，真正能把cookie设置上去的只有domain是.baidu.com的cookie绑定到了域名上，所以上面所说的不可跨域性，就是不能在不同的域名下用，每个cookie都会绑定单一的域名。

## cookie的格式

JS原生的API提供了获取`cookie`的方法：`document.cookie`（注意，这个方法只能获取非`HttpOnly`类型的`cookie`）。在`console`中执行这段代码可以看到结果如下图：

![cookie格式](https://image-static.segmentfault.com/124/862/1248624194-56dd2f47053e4_articlex)

## `cookie`属性

每个`cookie`都有一定的属性，如什么时候失效，要发送到哪个域名，哪个路径等等。这些属性是通过`cookie`选项来设置的，`cookie`选项包括：`expires`、`domain`、`path`、`secure`、`HttpOnly`。在设置任一个`cookie`时都可以设置相关的这些属性，当然也可以不设置，这时会使用这些属性的默认值。在设置这些属性时，属性之间由一个分号和一个空格隔开。代码示例如下：

```javascript
"key=name; expires=Thu, 25 Feb 2016 04:18:00 GMT; domain=ppsc.sankuai.com; path=/; secure; HttpOnly"
```

![cookie属性](https://user-gold-cdn.xitu.io/2017/10/2/88a294c5374093cedd41bb1ce50cd9d4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

* `name`
* `value`
* `domain`
* `path`
* `expires`
* `secure`
* `HttpOnly`

### `name`

这个显而易见，就是代表cookie的名字的意思，一个域名下绑定的cookie，name不能相同，相同的name的值会被覆盖掉，有兴趣的同学可以试一试，我在项目中切实用到过。

### `value`

这个就是每个cookie拥有的一个属性，它表示cookie的值，但是我在这里想说的不是这个，因为我在网上看到两种说法，如下：

1. cookie的值必须被URL编码
2. 对cookie的值进行编码不是必须的，还举了原始文档中所说的，仅对三种符号必须进行编码：分号、逗号和空格

这个东西得一分为二来看，先看下面的图：

![value](https://user-gold-cdn.xitu.io/2017/10/2/fb6f3ec85759285e8e9eb57fd979078b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

我在网上看到那么一种说法：

> 由于cookie规定是名称/值是不允许包含分号，逗号，空格的，所以为了不给用户到来麻烦，考虑服务器的兼容性，任何存储cookie的数据都应该被编码。

### `domain`和`path`

`domain`是域名，`path`是路径，两者加起来就构成了 URL，`domain`和`path`一起来限制`cookie`能被哪些 URL 访问。

一句话概括：某`cookie`的`domain`为`“baidu.com”`, `path`为`“/ ”`，若请求的URL(URL可以是`js/html/img/css`资源请求，但不包括`XHR`请求)的域名是`“baidu.com”`或其子域如`“api.baidu.com”`、`“dev.api.baidu.com”`，且 URL 的路径是`“/ ”`或子路径`“/home”`、`“/home/login”`，则浏览器会将此 `cookie` 添加到该请求的 `cookie` 头部中。

所以`domain`和`path`2个选项共同决定了`cookie`何时被浏览器自动添加到请求头部中发送出去。如果没有设置这两个选项，则会使用默认值。`domain`的默认值为设置该`cookie`的网页所在的域名，`path`默认值为设置该`cookie`的网页所在的目录。

> 特别说明1：
> 发生跨域xhr请求时，即使请求URL的域名和路径都满足 `cookie` 的 `domain` 和 `path`，默认情况下`cookie`也不会自动被添加到请求头部中。若想知道原因请阅读本文最后一节）
>
>特别说明2：
> `domain`是可以设置为页面本身的域名（本域），或页面本身域名的父域，但不能是公共后缀`public suffix`。举例说明下：如果页面域名为`www.baidu.com`, `domain`可以设置为`“www.baidu.com”`，也可以设置为`“baidu.com”`，但不能设置为`“.com”`或`“com”`。

### `expires`

![expires](https://user-gold-cdn.xitu.io/2017/10/2/348aab52f1892d29ecd3a1e5e5167cb9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

`expires`选项用来设置“`cookie`什么时间内有效”。

`expires`其实是`cookie`失效日期，`expires`必须是`GMT`格式的时间（可以通过`new Date().toGMTString()`或者`new Date().toUTCString()`来获得）。如`expires=Thu, 25 Feb 2016 04:18:00 GMT`表示`cookie`讲在2016年2月25日4:18分之后失效，对于失效的`cookie`浏览器会清空。

如果没有设置该选项，则默认有效期为`session`，即会话`cookie`。这种`cookie`在浏览器关闭后就没有了。

> expires 是 http/1.0协议中的选项，在新的http/1.1协议中expires已经由 max-age 选项代替，两者的作用都是限制cookie 的有效时间。expires的值是一个时间点（cookie失效时刻= expires），而max-age 的值是一个以秒为单位时间段（cookie失效时刻= 创建时刻+ max-age）。
> 另外，max-age 的默认值是 -1(即有效期为 session )；若max-age有三种可能值：负数、0、正数。负数：有效期session；0：删除cookie；正数：有效期为创建时刻+ max-age

如果你想要`cookie`存在一段时间，那么你可以通过设置`Expires`属性为未来的一个时间节点，`Expires`这个是代表当前时间的，这个属性已经逐渐被我们下面这个主人公所取代——Max-Age。

Max-Age，是以秒为单位的。

* Max-Age为正数时，`cookie`会在Max-Age秒之后，被删除。
* 当Max-Age为负数时，表示的是临时储存，不会生出`cookie`文件，只会存在浏览器内存中，且只会在打开的浏览器窗口或者子窗口有效，一旦浏览器关闭，`cookie`就会消失。
* 当Max-Age为0时，又会发生什么呢，删除`cookie`，因为`cookie`机制本身没有设置删除`cookie`，失效的`cookie`会被浏览器自动从内存中删除，所以，它实现的就是让`cookie`失效。

### `secure`

![secure](https://user-gold-cdn.xitu.io/2017/10/2/dd48df2362163b22d8d69d21918d8835?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

这个属性译为安全，http不仅是无状态的，还是不安全的协议，容易被劫持，打个比方，你在手机端浏览网页的时候，有没有中国移动图标跳出来过，闲言少叙，当这个属性设置为true时，此cookie只会在https和ssl等安全协议下传输

**提示：** *这个属性并不能对客户端的cookie进行加密，不能保证绝对的安全性。*

### `HttpOnly`

这个属性是面试的时候常考的，如果这个属性设置为true，就不能通过js脚本来获取cookie的值，能有效的防止xss攻击,看MDN的官方文档：

![HttpOnly](https://user-gold-cdn.xitu.io/2017/10/2/cf582beb568a79b1d9dd8ef97be707f6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> ——httpOnly与安全
>
> 从上面介绍中，大家是否会有这样的疑问：为什么我们要限制客户端去访问cookie？其实这样做是为了保障安全。
>
> 试想：如果任何 cookie 都能被客户端通过document.cookie获取会发生什么可怕的事情。当我们的网页遭受了 XSS 攻击，有一段恶意的script脚本插到了网页中。这段script脚本做的事情是：通过document.cookie读取了用户身份验证相关的 cookie，并将这些 cookie 发送到了攻击者的服务器。攻击者轻而易举就拿到了用户身份验证信息，于是就可以摇摇大摆地冒充此用户访问你的服务器了（因为攻击者有合法的用户身份验证信息，所以会通过你服务器的验证）。

## 关于js操作`cookie`

document.cookie可以对cookie进行读写，看一下两条指令：

```javascript
//读取浏览器中的cookie
console.log(document.cookie);
//写入cookie
document.cookie="age=12; expires=Thu, 26 Feb 2116 11:50:25 GMT; domain=sankuai.com; path=/";
```

注意：

* 客户端可以设置cookie 的下列选项：`expires`、`domain`、`path`、`secure`（有条件：只有在`https`协议的网页中，客户端设置`secure`类型的`cookie`才能成功），但无法设置`HttpOnly`选项。

### 设置多个`cookie`

当要设置多个`cookie`时， js代码很自然地我们会这么写：

```javascript
document.cookie = "name=Jonh; age=12; class=111";
```

但你会发现这样写只是添加了第一个`cookie``“name=John”`，后面的所有`cookie`都没有添加成功。所以最简单的设置多个`cookie`的方法就在重复执行`document.cookie = "key=name"`，如下：

```javascript
document.cookie = "name=Jonh";
document.cookie = "age=12";
document.cookie = "class=111";
```

## 服务端如何设置`cookie`

关于怎么设置cookie，我们只要打开控制台，看一个http的请求头和响应头中的东西即可明白：

![服务端如何设置cookie](https://user-gold-cdn.xitu.io/2017/10/2/ac1f0d4e46b21da20d76b8136dd7583f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

不管你是请求一个资源文件（如`html/js/css/image`），还是发送一个`ajax`请求，服务端都会返回`response`。而`response header`中有一项叫`set-cookie`，是服务端专门用来设置`cookie`的。如下图所示，服务端返回的`response header`中有5个`set-cookie`字段，每个字段对应一个`cookie`（注意不能将多个`cookie`放在一个`set-cookie`字段中），`set-cookie`字段的值就是普通的字符串，每个`cookie`还设置了相关属性选项。

注意：

* 一个`Set-Cookie`字段只能设置一个`cookie`，当你要想设置多个 `cookie`，需要添加同样多的`Set-Cookie`字段。
* 服务端可以设置`cookie` 的所有选项：`expires`、`domain`、`path`、`secure`、`HttpOnly`

## 我们看到的`cookie`

发送一个AJAX请求，`header`如下图：

![我们看到的cookie](https://image-static.segmentfault.com/239/160/2391604247-56dd2eafae06e_articlex)

从上图中我们会看到`request header`中自动添加了`Cookie`字段（我并没有手动添加这个字段哦~），`Cookie`字段的值其实就是我设置的那4个`cookie`。这个请求最终会发送到`http://ppsc.sankuai.com`这个服务器上，这个服务器就能从接收到的`request header`中提取那4个`cookie`。

上图展示了`cookie`的基本通信流程：设置`cookie` => `cookie`被自动添加到`request header`中 => 服务端接收到`cookie`。这个流程中有几个问题需要好好研究：

1. 什么样的数据适合放在`cookie`中？
2. `cookie`是怎么设置的？
3. `cookie`为什么会自动加到`request header`中？
4. `cookie`怎么增删查改？

我们要带着这几个问题继续往下阅读。

## `cookie`是怎么工作的？

首先必须明确一点，存储`cookie`是浏览器提供的功能。`cookie`其实是存储在浏览器中的纯文本，浏览器的安装目录下会专门有一个`cookie`文件夹来存放各个域下设置的`cookie`。

当网页要发`http`请求时，浏览器会先检查是否有相应的`cookie`，有则自动添加在`request header`中的`cookie`字段中。这些是浏览器自动帮我们做的，而且每一次`http`请求浏览器都会自动帮我们做。这个特点很重要，因为这关系到“什么样的数据适合存储在`cookie`中”。

存储在`cookie`中的数据，每次都会被浏览器自动放在`http`请求中，如果这些数据并不是每个请求都需要发给服务端的数据，浏览器这设置自动处理无疑增加了网络开销；但如果这些数据是每个请求都需要发给服务端的数据（比如身份认证信息），浏览器这设置自动处理就大大免去了重复添加操作。所以对于那设置“每次请求都要携带的信息（最典型的就是身份认证信息）”就特别适合放在`cookie`中，其他类型的数据就不适合了。

但在`localStorage`出现之前，`cookie`被滥用当做了存储工具。什么数据都放在`cookie`中，即使这些数据只在页面中使用而不需要随请求传送到服务端。当然`cookie`标准还是做了一些限制的：每个域名下的`cookie`的大小最大为`4KB`，每个域名下的`cookie`数量最多为`20`个（但很多浏览器厂商在具体实现时支持大于`20`个）。

## 如何修改、删除

### 修改`cookie`

要想修改一个`cookie`，只需要重新赋值就行，旧的值会被新的值覆盖。但要注意一点，在设置新`cookie`时，`path/domain`这几个选项一定要和旧`cookie`保持一样。否则不会修改旧值，而是添加了一个新的 `cookie`。

### 删除 `cookie`

删除一个`cookie`也挺简单，也是重新赋值，只要将这个新`cookie`的`expires`选项设置为一个过去的时间点就行了。但同样要注意，`path/domain`这几个选项一定要旧`cookie`保持一样。

## `cookie` 编码

`cookie`其实是个字符串，但这个字符串中逗号、分号、空格被当做了特殊符号。所以当`cookie`的`key`和`value`中含有这3个特殊字符时，需要对其进行额外编码，一般会用`escape`进行编码，读取时用`unescape`进行解码；当然也可以用`encodeURIComponent/decodeURIComponent`或者`encodeURI/decodeURI`（[三者的区别可以参考这篇文章](http://www.cnblogs.com/season-huang/p/3439277.html)）。

```javascript
var key = escape("name;value");
var value = escape("this is a value contain , and ;");
document.cookie= key + "=" + value + "; expires=Thu, 26 Feb 2116 11:50:25 GMT; domain=sankuai.com; path=/";
```

## 跨域请求中 `cookie`

之前在介绍 XHR 的一篇文章里面提过：默认情况下，在发生跨域时，`cookie`作为一种`credential`信息是不会被传送到服务端的。必须要进行额外设置才可以。具体原因和如何设置可以参考我的这篇文章：[你真的会使用XMLHttpRequest吗？](https://segmentfault.com/a/1190000004322487#articleHeader13)

另外，关于跨域资源共享 CORS极力推荐大家阅读阮一峰老师的这篇 跨域资源共享 CORS 详解。

## 其他补充

1. 什么时候 `cookie` 会被覆盖：`name/domain/path`这3个字段都相同的时候；
2. 关于domain的补充说明（[参考1](https://tools.ietf.org/html/rfc6265#section-5.2.3)/[参考2](http://erik.io/blog/2014/03/04/definitive-guide-to-cookie-domains/)）：
  1. 如果显式设置了 domain，则设置成什么，浏览器就存成什么；但如果没有显式设置，则浏览器会自动取 url 的 host 作为 domain 值；
  2. 新的规范中，显式设置 domain 时，如果 value 最前面带点，则浏览器处理时会将这个点去掉，所以最后浏览器存的就是没有点的（注意：但目前大多数浏览器并未全部这么实现）
  3. 前面带点‘.’和不带点‘.’有啥区别：
    * 带点：任何 subdomain 都可以访问，包括父 domain
    * 不带点：只有完全一样的域名才能访问，subdomain 不能（但在 IE 下比较特殊，它支持 subdomain 访问）

## 总结

咱们今天就聊到这里，若有不对之处欢迎各位指正~~
最后附上一些参考资料：

* [http://www.quirksmode.org/js/cookies.html](http://www.quirksmode.org/js/cookies.html)
* [http://www.tutorialspoint.com/javascript/javascript_cookies.htm](http://www.tutorialspoint.com/javascript/javascript_cookies.htm)
* [http://www.allaboutcookies.org/cookies/cookies-the-same.html](http://www.allaboutcookies.org/cookies/cookies-the-same.html)
* [http://bubkoo.com/2014/04/21/http-cookies-explained/](http://bubkoo.com/2014/04/21/http-cookies-explained/)
