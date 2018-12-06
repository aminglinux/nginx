####Nginx访问控制 —— deny_allow
```
Nginx的deny和allow指令是由ngx_http_access_module模块提供，Nginx安装默认内置了该模块。
除非在安装时有指定 --without-http_access_module。
```

#####语法
```
语法：allow/deny address | CIDR | unix: | all

它表示，允许/拒绝某个ip或者一个ip段访问.如果指定unix:,那将允许socket的访问。
注意：unix在1.5.1中新加入的功能。

在nginx中，allow和deny的规则是按顺序执行的。

```

#####示例
```
示例1：
location /
{
    allow 192.168.0.0/24;
    allow 127.0.0.1;
    deny all;
}

说明：这段配置值允许192.168.0.0/24网段和127.0.0.1的请求，其他来源IP全部拒绝。

示例2：
location ~ "admin"
{
    allow 110.21.33.121;
    deny all
}
说明：访问的uri中包含admin的请求，只允许110.21.33.121这个IP的请求。

```

