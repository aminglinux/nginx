####Nginx正向代理配置

    Nginx正向代理使用场景并不多见。
    需求场景1：
    如果在机房中，只有一台机器可以联网，其他机器只有内网，内网的机器想用使用yum安装软件包，在能能联网的机器上配置一个正向代理即可。
    
#####Nginx正向代理配置文件

```
server {
    listen 80 default_server;
    resolver 119.29.29.29;
    location /
    {
        proxy_pass http://$host$request_uri;
    }
}
```

#####Nginx正向代理配置执行说明

* resolver


```    
语法：resolver address;

address为DNS服务器的地址，国内通用的DNS 119.29.29.29为dnspod公司提供。 国际通用DNS 8.8.8.8或者8.8.4.4为google提供。
其他可以参考 http://dns.lisect.com/
    
示例：resolver 119.29.29.29;
```

* default_server

```
之所以要设置为默认虚拟主机，是因为这样就不用设置server_name了，任何域名解析过来都可以正常访问。
```
    
* proxy_pass

```
该指令用来设置要代理的目标url，正向代理服务器设置就保持该固定值即可。关于该指令的详细解释在反向代理配置中。
```