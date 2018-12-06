####Nginx的负载均衡配置
```
Nginx通过upstream和proxy_pass实现了负载均衡。本质上也是Nginx的反向代理功能，只不过后端的server为多个。
```

#####案例一（简单的轮询）
```
upstream www {
    server 172.37.150.109:80;
    server 172.37.150.101:80;
    server 172.37.150.110:80;
}

server {
    listen 80;
    server_name www.aminglinux.com;
    location / {
        proxy_pass http://www/;
        proxy_set_header Host   $host;
        proxy_set_header X-Real-IP      $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

说明：当被代理的机器有多台时，需要使用upstream来定义一个服务器组，
其中www名字可以自定义，在后面的proxy_pass那里引用。
这样nginx会将请求均衡地轮询发送给www组内的三台服务器。

```

#####案例二（带权重轮询+ip_hash算法）
```
upstream www {
    server 172.37.150.109:80 weight=50;
    server 172.37.150.101:80 weight=100;
    server 172.37.150.110:80 weight=50;
    ip_hash;
}

server {
    listen 80;
    server_name www.aminglinux.com;
    location / {
        proxy_pass http://www/;
        proxy_set_header Host   $host;
        proxy_set_header X-Real-IP      $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

说明：可以给www组内的三台机器配置权重，权重越高，则分配到的请求越多。
ip_hash为nginx负载均衡算法，原理很简单，它根据请求所属的客户端IP计算得到一个数值，然后把请求发往该数值对应的后端。
所以同一个客户端的请求，都会发往同一台后端，除非该后端不可用了。ip_hash能够达到保持会话的效果。

```

#####案例三（upstream其他配置）
```
upstream www {
        server 172.37.150.109:80 weight=50 max_fails=3 fail_timeout=30s;
        server 172.37.150.101:80 weight=100;
        server 172.37.150.110:80 down;
        server 172.37.150.110:80 backup;
}
server
{
    listen 80;
    server_name www.aminglinux.com;
    location / {
        proxy_next_upstream off;
        proxy_pass http://www/;
        proxy_set_header Host   $host;
        proxy_set_header X-Real-IP      $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

说明：down，表示当前的server不参与负载均衡；
backup，为预留的机器，当其他的server（非backup）出现故障或者忙的时候，才会请求backup机器;
max_fails，允许请求失败的次数，默认为1。当失败次数达到该值，就认为该机器down掉了。 失败的指标是由proxy_next_upstream模块定义，其中404状态码不认为是失败。
fail_timeount，定义失败的超时时间，也就是说在该时间段内达到max_fails，才算真正的失败。默认是10秒。

proxy_next_upstream，通过后端服务器返回的响应状态码，表示服务器死活，可以灵活控制后端机器是否加入分发列表。
语法: proxy_next_upstream error | timeout | invalid_header | http_500 | http_502 | http_503 | http_504 |http_404 | off ...; 
默认值: proxy_next_upstream error timeout

error      # 和后端服务器建立连接时，或者向后端服务器发送请求时，或者从后端服务器接收响应头时，出现错误
timeout    # 和后端服务器建立连接时，或者向后端服务器发送请求时，或者从后端服务器接收响应头时，出现超时
invalid_header  # 后端服务器返回空响应或者非法响应头
http_500   # 后端服务器返回的响应状态码为500
http_502   # 后端服务器返回的响应状态码为502
http_503   # 后端服务器返回的响应状态码为503
http_504   # 后端服务器返回的响应状态码为504
http_404   # 后端服务器返回的响应状态码为404
off        # 停止将请求发送给下一台后端服务器

```

#####案例四（根据不同的uri）
```
    upstream aa.com {         
                      server 192.168.0.121;
                      server 192.168.0.122;  
     }
    upstream bb.com {  
                       server 192.168.0.123;
                       server 192.168.0.124;
    }
    server {
        listen       80;
        server_name  www.aminglinux.com;
        location ~ aa.php
        {
            proxy_pass http://aa.com/;
            proxy_set_header Host   $host;
            proxy_set_header X-Real-IP      $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        location ~ bb.php
        {
              proxy_pass http://bb.com/;
              proxy_set_header Host   $host;
              proxy_set_header X-Real-IP      $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        location /
        {
              proxy_pass http://bb.com/;
              proxy_set_header Host   $host;
              proxy_set_header X-Real-IP      $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}

说明：请求aa.php的，会到aa.com组，请求bb.php的会到bb.com，其他请求全部到bb.com。

```
#####案例五（根据不同的目录）
```
upstream aaa.com
{
            server 192.168.111.6;
}
upstream bbb.com
{
            server 192.168.111.20;
}
server {
        listen 80;
        server_name www.aminglinux.com;
        location /aaa/
        {
            proxy_pass http://aaa.com/aaa/;
            proxy_set_header Host   $host;
            proxy_set_header X-Real-IP      $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        location /bbb/
        {
            proxy_pass http://bbb.com/bbb/;
            proxy_set_header Host   $host;
            proxy_set_header X-Real-IP      $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        location /
        {
            proxy_pass http://bbb.com/;
            proxy_set_header Host   $host;
            proxy_set_header X-Real-IP      $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}
```