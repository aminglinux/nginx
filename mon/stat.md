####配置Nginx状态
```
Nginx有内置一个状态页，需要在编译的时候指定参数--with-http_stub_status_module参数方可打开。
也就是说，该功能是由http_stub_status_module模块提供，默认没有加载。
```

#####Nginx配置文件示例
```
server{
	listen 80;
	server_name www.aminglinux.com;
	
	location /status/ {
	    stub_status on;
	    access_log off;
	    allow 127.0.0.1;
	    allow 192.168.10.0/24;
	    deny all;
	}
}

```

#####配置说明
* location /status/这样当访问/status/时即可访问到状态页内容。
* stub_status on即打开了状态页。
* access_log off不记录日志
* allow和deny只允许指定IP和IP段访问，因为这个页面需要保护起来，并不公开，当然也可以做用户认证。

#####测试和结果说明
```
测试命令：curl -x127.0.0.1:80 www.aminglinux.com/status/

结果如下：
Active connections: 1 
server accepts handled requests
 11 11 11 
Reading: 0 Writing: 1 Waiting: 0 

说明：
active connections – 活跃的连接数量
server accepts handled requests — 总共处理的连接数、成功创建的握手次数、总共处理的请求次数
需要注意，一个连接可以有多次请求。
reading — 读取客户端的连接数.
writing — 响应数据到客户端的数量
waiting — 开启 keep-alive 的情况下,这个值等于 active – (reading+writing), 意思就是 Nginx 已经处理完正在等候下一次请求指令的驻留连接.
```