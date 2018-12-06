####Nginx+Tomcat架构
```
配置文件示例
server
{
    listen 80;
    server_name www.aminglinux.com;
    
    location ~* "\.(jpg|png|jepg|js|css|xml|bmp|swf|gif|html)$"
    {
        root /data/wwwroot/aminglinux/;
        access_log off;
        expire 7d;
    }
    
    location /
    {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP      $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

说明：
1 首先，把各种静态文件的请求分离出来，单独由nginx处理。
2 其他请求直接代理8080端口，即tomcat服务。
```