#### nginx 从1.9.0版本开始支持四层代理，但做四层代理时 编译需要添加  --with-stream模块

#### nginx配置文件示例

```
http {
    ......
}

## stram模块 和http模块是一同等级
stream {    
      upstream app_server{
          server 192.168.0.14:8028;
          #server 192.168.0.15:8028;
      }
      
      server {
          listen 8028;                          #8028端口将以4层TCP协议方式转发至后端app_sever;
          proxy_pass app_server;
      }

}
```

#### UDP示例
```
stream {         
    upstream dns {                 
        server 192.168.0.13:53;                      
    }          
    
    server {                 
        listen 53 udp;                 
        proxy_pass dns;                 
        proxy_timeout 5s;                     
    } 
}

```
