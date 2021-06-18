#### Nginx正向代理配置

    Nginx正向代理使用场景并不多见。
    需求场景1：
    如果在机房中，只有一台机器可以联网，其他机器只有内网，内网的机器想用使用yum安装软件包，在能能联网的机器上配置一个正向代理即可。
    
#####  Nginx正向代理配置文件

说明： 以下配置文件为nginx官方提供，该方法只能实现针对http的网站的访问，如果是https就会有问题。要想实现https的正向代理，可以使用一个三方模块，后面介绍。
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

#####  Nginx正向代理配置执行说明

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

#####  Nginx正向代理支持https

下载三方模块ngx_http_proxy_connect_module，github地址：https://github.com/chobits/ngx_http_proxy_connect_module
注意，不同的Nginx版本，还需要下载不同的patch包。

下面的例子，以1.9.12为例

```
wget http://nginx.org/download/nginx-1.9.12.tar.gz
tar -xzvf nginx-1.9.12.tar.gz
cd nginx-1.9.12/
patch -p1 < /path/to/ngx_http_proxy_connect_module/patch/proxy_connect.patch
./configure --add-dynamic-module=/path/to/ngx_http_proxy_connect_module
make && make install

```

Nginx正向代理配置文件
```
server {
     listen                         3128;


     # dns resolver used by forward proxying
     resolver                       119.29.29.29;


     # forward proxy for CONNECT request
     proxy_connect;
     proxy_connect_allow            80 443 3000 9070 9074;
     proxy_connect_connect_timeout  10s;
     proxy_connect_read_timeout     10s;
     proxy_connect_send_timeout     10s;


     # forward proxy for non-CONNECT request
     location / {
         proxy_pass http://$host;
         proxy_set_header Host $host;
     }
}

```

测试示例
```
$ curl https://github.com/ -v -x 127.0.0.1:3128
*   Trying 127.0.0.1...                                           -.
* Connected to 127.0.0.1 (127.0.0.1) port 3128 (#0)                | curl creates TCP connection with nginx (with proxy_connect module).
* Establish HTTP proxy tunnel to github.com:443                   -'
> CONNECT github.com:443 HTTP/1.1                                 -.
> Host: github.com:443                                         (1) | curl sends CONNECT request to create tunnel.
> User-Agent: curl/7.43.0                                          |
> Proxy-Connection: Keep-Alive                                    -'
>
< HTTP/1.0 200 Connection Established                             .- nginx replies 200 that tunnel is established.
< Proxy-agent: nginx                                           (2)|  (The client is now being proxied to the remote host. Any data sent
<                                                                 '-  to nginx is now forwarded, unmodified, to the remote host)

* Proxy replied OK to CONNECT request
* TLS 1.2 connection using TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256  -.
* Server certificate: github.com                                   |
* Server certificate: DigiCert SHA2 Extended Validation Server CA  | curl sends "https://github.com" request via tunnel,
* Server certificate: DigiCert High Assurance EV Root CA           | proxy_connect module will proxy data to remote host (github.com).
> GET / HTTP/1.1                                                   |
> Host: github.com                                             (3) |
> User-Agent: curl/7.43.0                                          |
> Accept: */*                                                     -'
>
< HTTP/1.1 200 OK                                                 .-
< Date: Fri, 11 Aug 2017 04:13:57 GMT                             |
< Content-Type: text/html; charset=utf-8                          |  Any data received from remote host will be sent to client
< Transfer-Encoding: chunked                                      |  by proxy_connect module.
< Server: GitHub.com                                           (4)|
< Status: 200 OK                                                  |
< Cache-Control: no-cache                                         |
< Vary: X-PJAX                                                    |
...                                                               |
... <other response headers & response body> ...                  |
...                                                               

```

#### linux机器上配置全局代理
在 /etc/profile 文件中增加如下三项。  
```
export proxy="http://{proxy_server_ip}:3128"
export http_proxy=$proxy
export https_proxy=$proxy
```

使配置生效
```
source /etc/profile
```

对那些没有域名解析通过绑定hosts文件来访问的域名，不让其走http/https代理，需要额外增加环境变量：
```
export no_proxy='a.test.com,127.0.0.1,2.2.2.2'
```
