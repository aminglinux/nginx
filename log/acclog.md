####Nginx访问日志配置
```
web服务器的访问日志是非常重要的，我们可以通过访问日志来分析用户的访问情况，
也可以通过访问日志发现一些异常访问，比如cc攻击。

格式： access_log /path/to/logfile format;

access_log可以配置到http, server, location配置段中。

```
#####配置示例
```
server 
{
    listen 80;
    server_name www.aminglinux.com;
    root /data/wwwroot/www.aminglinux.com;
    index index.html index.php;
    access_log /data/logs/www.aminglinux.com_access.log main;
}
说明：若不指定log_format，则按照默认的格式写日志。

```
