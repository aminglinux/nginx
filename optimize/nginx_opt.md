####Nginx配置参数优化

```
Nginx作为高性能web服务器，即使不特意调整配置参数也可以处理大量的并发请求。
以下的配置参数是借鉴网上的一些调优参数，仅作为参考，不见得适于你的线上业务。
```

#####worker进程

* worker_processes
```
该参数表示启动几个工作进程，建议和本机CPU核数保持一致，每一核CPU处理一个进程。
```

* worker_rlimit_nofile
```
它表示Nginx最大可用的文件描述符个数，需要配合系统的最大描述符，建议设置为102400。
还需要在系统里执行ulimit -n 102400才可以。
也可以直接修改配置文件/etc/security/limits.conf修改
增加：
#* soft nofile 655350 (去掉前面的#)
#* hard nofile 655350 (去掉前面的#)
```
* worker_connections 
```
该参数用来配置每个Nginx worker进程最大处理的连接数，这个参数也决定了该Nginx服务器最多能处理多少客户端请求
（worker_processes * worker_connections)，建议把该参数设置为10240，不建议太大。
```

#####http和tcp连接

* use epoll
```
使用epoll模式的事件驱动模型，该模型为Linux系统下最优方式。
```
* multi_accept on
```
使每个worker进程可以同时处理多个客户端请求。
```
* sendfile on
```
使用内核的FD文件传输功能，可以减少user mode和kernel mode的切换，从而提升服务器性能。
```
* tcp_nopush on
```
当tcp_nopush设置为on时，会调用tcp_cork方法进行数据传输。
使用该方法会产生这样的效果：当应用程序产生数据时，内核不会立马封装包，而是当数据量积累到一定量时才会封装，然后传输。
```
* tcp_nodelay on
```
不缓存data-sends（关闭 Nagle 算法），这个能够提高高频发送小数据报文的实时性。
(关于Nagle算法)
【假如需要频繁的发送一些小包数据，比如说1个字节，以IPv4为例的话，则每个包都要附带40字节的头，
也就是说，总计41个字节的数据里，其中只有1个字节是我们需要的数据。
为了解决这个问题，出现了Nagle算法。
它规定：如果包的大小满足MSS，那么可以立即发送，否则数据会被放到缓冲区，等到已经发送的包被确认了之后才能继续发送。
通过这样的规定，可以降低网络里小包的数量，从而提升网络性能。】
```

* keepalive_timeout
```
定义长连接的超时时间，建议30s，太短或者太长都不一定合适，当然，最好是根据业务自身的情况来动态地调整该参数。
```
* keepalive_requests
```
定义当客户端和服务端处于长连接的情况下，每个客户端最多可以请求多少次，可以设置很大，比如50000.
```
* reset_timeout_connection on
```
设置为on的话，当客户端不再向服务端发送请求时，允许服务端关闭该连接。
```
* client_body_timeout 
```
客户端如果在该指定时间内没有加载完body数据，则断开连接，单位是秒，默认60，可以设置为10。
```
* send_timeout
```
这个超时时间是发送响应的超时时间，即Nginx服务器向客户端发送了数据包，但客户端一直没有去接收这个数据包。
如果某个连接超过send_timeout定义的超时时间，那么Nginx将会关闭这个连接。单位是秒，可以设置为3。
```

#####buffer和cache(以下配置都是针对单个请求)

* client_body_buffer_size
```
当客户端以POST方法提交一些数据到服务端时，会先写入到client_body_buffer中，如果buffer写满会写到临时文件里，建议调整为128k。
```
* client_max_body_size
```
浏览器在发送含有较大HTTP body的请求时，其头部会有一个Content-Length字段，client_max_body_size是用来限制Content-Length所示值的大小的。
这个限制body的配置不用等Nginx接收完所有的HTTP包体，就可以告诉用户请求过大不被接受。会返回413状态码。
例如，用户试图上传一个1GB的文件，Nginx在收完包头后，发现Content-Length超过client_max_body_size定义的值，
就直接发送413(Request Entity Too Large)响应给客户端。
将该数值设置为0，则禁用限制，建议设置为10m。
```
* client_header_buffer_size
```
设置客户端header的buffer大小，建议4k。
```
* large_client_header_buffers
```
对于比较大的header（超过client_header_buffer_size）将会使用该部分buffer，两个数值，第一个是个数，第二个是每个buffer的大小。
建议设置为4 8k
```
* open_file_cache 
```
该参数会对以下信息进行缓存：
打开文件描述符的文件大小和修改时间信息;
存在的目录信息;
搜索文件的错误信息（文件不存在无权限读取等信息）。
格式：open_file_cache max=size inactive=time;
max设定缓存文件的数量，inactive设定经过多长时间文件没被请求后删除缓存。
建议设置 open_file_cache max=102400 inactive=20s;
```

* open_file_cache_valid
```
指多长时间检查一次缓存的有效信息。建议设置为30s。
```

* open_file_cache_min_uses
```
open_file_cache指令中的inactive参数时间内文件的最少使用次数，
如,将该参数设置为1，则表示，如果文件在inactive时间内一次都没被使用，它将被移除。
建议设置为2。
```

#####压缩

```
对于纯文本的内容，Nginx是可以使用gzip压缩的。使用压缩技术可以减少对带宽的消耗。
由ngx_http_gzip_module模块支持

配置如下：
gzip on; //开启gzip功能
gzip_min_length 1024; //设置请求资源超过该数值才进行压缩，单位字节
gzip_buffers 16 8k; //设置压缩使用的buffer大小，第一个数字为数量，第二个为每个buffer的大小
gzip_comp_level 6; //设置压缩级别，范围1-9,9压缩级别最高，也最耗费CPU资源
gzip_types text/plain application/x-javascript text/css application/xml image/jpeg image/gif image/png; //指定哪些类型的文件需要压缩
gzip_disable "MSIE 6\."; //IE6浏览器不启用压缩

测试：
curl -I -H "Accept-Encoding: gzip, deflate" http://www.aminglinux.com/1.css

```
#####日志
* 错误日志级别调高，比如crit级别，尽量少记录无关紧要的日志。
* 对于访问日志，如果不要求记录日志，可以关闭，
* 静态资源的访问日志关闭

#####静态文件过期
```
对于静态文件，需要设置一个过期时间，这样可以让这些资源缓存到客户端浏览器，
在缓存未失效前，客户端不再向服务期请求相同的资源，从而节省带宽和资源消耗。

配置示例如下：
location ~* ^.+\.(gif|jpg|png|css|js)$                                      
{
    expires 1d; //1d表示1天，也可以用24h表示一天。
}
```

#####作为代理服务器
```
Nginx绝大多数情况下都是作为代理或者负载均衡的角色。
因为前面章节已经介绍过以下参数的含义，在这里只提供对应的配置参数：
http
{
    proxy_cache_path /data/nginx_cache/ levels=1:2 keys_zone=my_zone:10m inactive=300s max_size=5g;
    ...;
    server
    {
	proxy_buffering on;
	proxy_buffer_size 4k;
	proxy_buffers 2 4k;
	proxy_busy_buffers_size 4k;
	proxy_temp_path /tmp/nginx_proxy_tmp 1 2;
	proxy_max_temp_file_size 20M;
	proxy_temp_file_write_size 8k;
	
	location /
	{
	    proxy_cache my_zone;
	    ...;
	}
    }
}

```

#####SSL优化
* 适当减少worker_processes数量，因为ssl功能需要使用CPU的计算。
* 使用长连接，因为每次建立ssl会话，都会耗费一定的资源（加密、解密）
* 开启ssl缓存，简化服务端和客户端的“握手”过程。
```
ssl_session_cache   shared:SSL:10m; //缓存为10M
ssl_session_timeout 10m; //会话超时时间为10分钟
```
