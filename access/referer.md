####Nginx基于$http_referer的访问控制
```
在前面讲解rewrite时，曾经用过该变量，当时实现了防盗链功能。
其实基于该变量，我们也可以做一些特殊的需求。

```

#####示例
```
背景：网站被黑挂马，搜索引擎收录的网页是有问题的，当通过搜索引擎点击到网站时，却显示一个博彩网站。
由于查找木马需要时间，不能马上解决，为了不影响用户体验，可以针对此类请求做一个特殊操作。
比如，可以把从百度访问的链接直接返回404状态码，或者返回一段html代码。

if ($http_referer ~ 'baidu.com')
{
    return 404;
}

或者

if ($http_referer ~ 'baidu.com')
{
    return 200 "<html><script>window.location.href='//$host$request_uri';</script></html>";
}
```