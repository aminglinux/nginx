####Nginx反向代理配置

```
Nginx反向代理在生产环境中使用很多的。

场景1：
域名没有备案，可以把域名解析到香港一台云主机上，在香港云主机做个代理，而网站数据是在大陆的服务器上。

示例1：
server 
{
    listen 80;
    server_name aminglinux.com;
    
    location /
    {
        proxy_pass http://123.23.13.11/;
        proxy_set_header Host   $host;
        proxy_set_header X-Real-IP      $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
#####配置说明
######1. proxy_pass

    在正向代理中，已经使用过该指令。
    格式很简单： proxy_pass  URL;
    其中URL包含：传输协议（http://, https://等）、主机名（域名或者IP:PORT）、uri。

    示例如下：
    proxy_pass http://www.aminglinux.com/;
    proxy_pass http://192.168.200.101:8080/uri;
    proxy_pass unix:/tmp/www.sock;
    
    对于proxy_pass的配置有几种情况需要注意。
    示例2：
    location /aming/
    {
        proxy_pass http://192.168.1.10;
        ...
    }
    
    
    示例3：
    location /aming/
    {
        proxy_pass http://192.168.1.10/;
        ...
    }
    
    示例4：
    location /aming/
    {
        proxy_pass http://192.168.1.10/linux/;
        ...
    }
    
    示例5：
    location /aming/
    {
        proxy_pass http://192.168.1.10/linux;
        ...
    }
    
    假设server_name为www.aminglinux.com
    当请求http://www.aminglinux.com/aming/a.html的时候，以上示例2-5分别访问的结果是
    
    示例2：http://192.168.1.10/aming/a.html
    
    示例3：http://192.168.1.10/a.html
    
    示例4：http://192.168.1.10/linux/a.html
    
    示例5：http://192.168.1.10/linuxa.html
    

######2. proxy_set_header
```
proxy_set_header用来设定被代理服务器接收到的header信息。

语法：proxy_set_header field value;
field为要更改的项目，也可以理解为变量的名字，比如host
value为变量的值

如果不设置proxy_set_header，则默认host的值为proxy_pass后面跟的那个域名或者IP（一般写IP），
比如示例4，请求到后端的服务器上时，完整请求uri为：http://192.168.1.10/linux/a.html

如果设置proxy_set_header，如 proxy_set_header host $host;
比如示例4，请求到后端的服务器完整uri为：http://www.aminglinux.com/linux/a.html

proxy_set_header X-Real-IP $remote_addr;和proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
用来设置被代理端接收到的远程客户端IP，如果不设置，则header信息中并不会透传远程真实客户端的IP地址。
可以用如下示例来测试：

示例6(被代理端)
server{
	listen 8080;
	server_name www.aminglinux.com;
	root /tmp/123.com_8080;
	index index.html;
        location /linux/ {
	    echo "$host";
	    echo $remote_addr;
	    echo $proxy_add_x_forwarded_for;
	}
}

示例7（代理服务器上）
server {
    listen 80;
    server_name www.aminglinux.com;

    location /aming/
    {
	proxy_pass http://192.168.1.10:8080/linux/;
	proxy_set_header host $host;
	proxy_set_header X-Real-IP      $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

```
######3. proxy_redirect
```
该指令用来修改被代理服务器返回的响应头中的Location头域和“refresh”头域。
语法结构为：
proxy_redirect redirect replacement;
proxy_redirect default;
proxy_redirect off;

示例8：
server {
    listen 80;
    server_name www.aminglinux.com;
    index  index.html;

    location /
    {
	proxy_pass http://127.0.0.1:8080;
	proxy_set_header host $host;
	proxy_set_header X-Real-IP      $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

当请求的链接为 http://www.aminglinux.com/aming
结果会返回301，定向到了 http://www.aminglinux.com:8080/aming/

注意：返回301有几个先决条件
1. location后面必须是/; 
2. proxy_pass后面的URL不能加uri,只能是IP或者IP:port结尾，并不能以/结尾；
3. 访问的uri必须是一个真实存在的目录，如，这里的aming必须是存在的
4. 访问的时候，不能以/结尾，只能是 www.aminglinux.com/aming

虽然，这4个条件挺苛刻，但确实会遇到类似的请求。解决方法是，加一行proxy_redirect http://$host:8080/ /;

示例9：
server {
    listen 80;
    server_name www.aminglinux.com;
    index  index.html;

    location /
    {
	proxy_pass http://127.0.0.1:8080;
	proxy_set_header host $host;
	proxy_redirect http://$host:8080/ /;
	proxy_set_header X-Real-IP      $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}


```