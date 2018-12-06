####Nginx基于$user_agent的访问控制
```
user_agent大家并不陌生，可以简单理解成浏览器标识，包括一些蜘蛛爬虫都可以通过user_agent来辨识。
通过观察访问日志，可以发现一些搜索引擎的蜘蛛对网站访问特别频繁，它们并不友好。
为了减少服务器的压力，其实可以把除主流搜索引擎蜘蛛外的其他蜘蛛爬虫全部封掉。
另外，一些cc攻击，我们也可以通过观察它们的user_agent找到规律。

```
#####示例
```
if ($user_agent ~ 'YisouSpider|MJ12bot/v1.4.2|YoudaoBot|Tomato')
{
    return 403;
}
说明：user_agent包含以上关键词的请求，全部返回403状态码。

测试：
1. curl -A "123YisouSpider1.0"
2. curl -A "MJ12bot/v1.4.1"

```

