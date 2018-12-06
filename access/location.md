####基于location的访问控制
```
在生产环境中，我们会对某些特殊的请求进行限制，比如对网站的后台进行限制访问。
这就用到了location配置。
```

#####示例1
```
location /aming/
{
    deny all;
}

说明：针对/aming/目录，全部禁止访问，这里的deny all可以改为return 403.
```

#####示例2
```
location ~ ".bak|\.ht"
{
    return 403;
}
说明：访问的uri中包含.bak字样的或者包含.ht的直接返回403状态码。

测试链接举例：
1. www.aminglinux.com/123.bak
2. www.aminglinux.com/aming/123/.htalskdjf
```

#####示例3
```
location ~ (data|cache|tmp|image|attachment).*\.php$
{
    deny all;
}

说明：请求的uri中包含data、cache、tmp、image、attachment并且以.php结尾的，全部禁止访问。

测试链接举例：
1. www.aminglinux.com/aming/cache/1.php
2. www.aminglinux.com/image/123.phps
3. www.aminglinux.com/aming/datas/1.php

```