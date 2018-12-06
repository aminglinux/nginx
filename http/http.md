####HTTP协议相关概念
HTTP协议是Hyper Text Transfer Protocol（超文本传输协议）的缩写，是用于从万维网（WWW:World Wide Web ）服务器传输超文本到本地浏览器的传送协议。 
HTTP 是基于 TCP/IP 协议的应用层协议。它不涉及数据包（packet）传输，主要规定了客户端和服务器之间的通信格式，默认使用80端口。

#####请求方法
这个方法指的是，客户端向服务端发起请求时的动作。
常见方法如下：

| HTTP方法      | 说明    |
| :--------   | :-----   | 
|GET      |最常见的方法，客户端向服务端请求某个资源，并返回实际主体，例如请求图片、html等 |
|POST| 客户端向服务端提交指定数据，例如提交表单|
|HEAD| 类似于get请求，只不过返回的响应中没有具体的内容，只有头部信息|
|PUT| 类似于POST，也是向服务端提交数据，不过PUT有指定数据存储位置|
|DELETE|请求服务器删除指定的页面|
|OPTIONS|获取服务端支持的方法，它还可以查看服务端的性能|
|TRACE|回显服务器收到的请求，主要用于测试或诊断|
|CONNECT|把请求连接转换到透明的TCP/IP通道|

#####HTTP状态码
客户端发送请求到服务端，然后服务端会回应结果给客户端，回应的数据会包含一个三位数字的状态码，用来标识该请求是否成功，比如是正常还是错误等。
HTTP/1.1中状态码可以分为五大类。
| 状态码      | 说明    |
| :--------   | :-----   | 
|1**|	信息，服务器收到请求，需要请求者继续执行操作|
|2**|	成功，操作被成功接收并处理|
|3**|	重定向，需要进一步的操作以完成请求|
|4**|	客户端错误，请求包含语法错误或无法完成请求|
|5**|	服务器错误，服务器在处理请求的过程中发生了错误|

以下是几个常见的状态码
| 状态码      | 英文名称|说明    |
| :--------   | :-----   | :-----   | 
|200|	OK|表示成功客户端成功接收到了服务端返回的数据，这是最常见的状态码|
|206|Partial Content|客户端发完请求后，服务端只是返回了部分数据，就会出现该状态码，例如当下载一个很大的文件时，在没有下载完成前就会出现该状态码|
|301|moved permanently|永久移动。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。用作域名重定向|
|302|Moved Temporarily|临时移动。与301类似，URI被移动到了一个新的地址，但资源只是临时被移动，无论是301还是302对于客户端来说结果是一样的，这两个状态码针对搜索引擎来说是有差异的，考虑SEO的话，要使用301|
|400|Bad Request|客户端请求语法错误，服务端无法理解|
|401|Unauthorized|服务端如果开启了用户认证，而客户端没有提供正确的验证信息就会返回该状态码|
|403|Forbidden|服务端不允许客户端访问|
|404|Not Found|客户端请求的资源不存在|
|413|Request Entity Too Large|客户端向服务端上传一个比较大的文件时，如果文件大小超过了服务端的限制就会出现该状态码|
|500|Internal Server Error|服务端出现了内部错误|
|502|Bad Gateway|服务器充当代理角色时，后端被代理的服务器不可用或者没有正常回应，如，在nginx+php-fpm的环境中，如果php-fpm服务出现故障，nginx就会出现该状态码|
|503|Service Unavailable|服务当前不可用，由于超载或系统维护，服务器暂时的无法处理客户端的请求，如，当nginx限速后，客户端请求超过限制就会收到该状态码|
|504|Gateway Time-out|和502类似，充当代理角色时，后端的服务期没有按时返回数据，超时了|

#####HTTP Request
从客户端发往服务端的HTTP报文叫做HTTP Request Message。
HTTP请求报文由请求行、请求头部（header)、空行、请求数据几个部分组成，
如下所示：

![img](https://coding.net/u/aminglinux/p/nginx/git/raw/master/http/http_request.jpg)

使用curl命令可以获取Request头信息 curl -v

示例：
```
GET /123.jpg HTTP/1.1 请求行
Host    img.123.com  请求头部
User-Agent  Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.106 Safari/537.36 请求头部
Accept  image/webp,image/*,*/*;q=0.8  请求头部
Referer http://www.aminglinux.com/  请求头部
Accept-Encoding gzip, deflate, sdch 请求头部
Accept-Language zh-CN,zh;q=0.8  请求头部
空行
请求数据
```

#####HTTP Response
从服务端回应的HTTP报文叫做HTTP Response Message。
HTTP响应也由四个部分组成，分别是：状态行、消息报头、空行和响应正文。
示例：
```
HTTP/1.1 200 OK  状态行
Server: nginx/1.12.2  消息报头
Date: Tue, 03 Jul 2018 07:28:58 GMT   消息报头
Content-Type: text/html; charset=UTF-8   消息报头
Connection: keep-alive    消息报头
X-Powered-By: PHP/5.6.10   消息报头
空行
响应正文
```

#####HTTP工作原理
* 客户端连接到Web服务器，一个HTTP客户端，通常是浏览器，与Web服务器的HTTP端口（默认为80）建立一个TCP套接字连接。

* 通过TCP套接字，客户端向Web服务器发送一个文本的请求报文。

* Web服务器解析请求，定位请求资源。服务器将资源复本写到TCP套接字，由客户端读取。

* 若connection模式为close，则服务器主动关闭TCP连接，客户端被动关闭连接，释放TCP连接;若connection 模式为keepalive，则该连接会保持一段时间，在该时间内可以继续接收请求。

* 客户端浏览器首先解析状态行，查看表明请求是否成功的状态代码。然后解析每一个响应头，响应头告知以下为若干字节的HTML文档和文档的字符集。客户端浏览器读取响应数据HTML，根据HTML的语法对其进行格式化，并在浏览器窗口中显示。

例如：在浏览器地址栏键入URL，按下回车之后会经历以下流程：

* 浏览器向 DNS 服务器请求解析该 URL 中的域名所对应的 IP 地址;

* 解析出 IP 地址后，根据该 IP 地址和默认端口 80，和服务器建立TCP连接;

* 浏览器发出读取文件(URL 中域名后面部分对应的文件)的HTTP 请求，该请求报文作为 TCP 三次握手的第三个报文的数据发送给服务器;

* 服务器对浏览器请求作出响应，并把对应的 html 文本发送给浏览器;

* 释放 TCP连接;

* 浏览器将该 html 文本并显示内容;

#####URI和URL
URI，是uniform resource identifier，统一资源标识符，用来唯一的标识一个资源。
Web上可用的每种资源如HTML文档、图像、视频片段、程序等都是用一个URI来定位的。
URI一般由三部组成：
* 访问资源的命名机制，如http://, ftp://, rsync://等
* 存放资源的主机名，如www.aminglinux.com
* 资源自身的名称，由路径表示，着重强调于资源，如html/01/2018/0701.html
* 一个完整的URI示例：http://www.aminglinux.com/html/01/2018/0701.html

URL是uniform resource locator，统一资源定位器，它是一种具体的URI，即URL可以用来标识一个资源，而且还指明了如何定位这个资源。
URL是Internet上用来描述信息资源的字符串，主要用在各种WWW客户程序和服务器程序上。
采用URL可以用一种统一的格式来描述各种信息资源。
URL一般由三部组成：
* 协议(或称为服务方式)
* 存有该资源的主机IP地址(有时也包括端口号)
* 主机资源的具体地址。如目录和文件名等

