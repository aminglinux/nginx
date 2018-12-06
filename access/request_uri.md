####nginx基于$request_uri访问控制
```
$request_uri比$docuemnt_uri多了请求的参数。
主要是针对请求的uri中的参数进行控制。
```

#####示例
```
if ($request_uri ~ "gid=\d{9,12}")
{
    return 403;
}

说明：\d{9,12}是正则表达式，表示9到12个数字，例如gid=1234567890就符号要求。

测试链接：
1. www.aminglinux.com/index.php?gid=1234567890&pid=111  匹配
2. www.aminglinux.com/gid=123  不匹配

背景知识：
曾经有一个客户的网站cc攻击，对方发起太多类似这样的请求：/read-123405150-1-1.html
实际上，这样的请求并不是正常的请求，网站会抛出一个页面，提示帖子不存在。
所以，可以直接针对这样的请求，return 403状态码。

```