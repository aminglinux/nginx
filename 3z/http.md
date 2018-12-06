###http配置部分

#####官方文档 http://nginx.org/en/docs/

#####参考链接： https://segmentfault.com/a/1190000012672431

#####参考链接： https://segmentfault.com/a/1190000002797601

#####参考链接：http的header https://kb.cnblogs.com/page/92320/


####MIME-Type
    include       mime.types;  //cat conf/mime.types
    定义nginx能识别的网络资源媒体类型（如，文本、html、js、css、流媒体等）
    
    default_type  application/octet-stream;
    定义默认的type，如果不定义改行，默认为text/plain.
    
####log_format 
    log_format main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    其中main为日志格式的名字，后面的为nginx的内部变量组成的一串字符串。
    
####access_log logs/access.log  main;

    定义日志的路径以及采用的日志格式，该参数可以在server配置块中定义。
    

####sendfile on;

    是否调用sendfile函数传输文件，默认为off，使用sendfile函数传输，可以减少user mode和kernel mode的切换，从而提升服务器性能。
    对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。
    
####sendfile_max_chunk 128k;

    该参数限定Nginx worker process每次调用sendfile()函数传输数据的最大值，默认值为0，如果设置为0则无限制。
    
####tcp_nopush on;

    当tcp_nopush设置为on时，会调用tcp_cork方法进行数据传输。
    使用该方法会产生这样的效果：当应用程序产生数据时，内核不会立马封装包，而是当数据量积累到一定量时才会封装，然后传输。这样有助于解决网络堵塞问题。
    默认值为on。举例：快递员收快递、发快递，包裹累积到一定量才会发，节省运输成本。

####keepalive_timeout  65 60;

    该参数有两个值，第一个值设置nginx服务器与客户端会话结束后仍旧保持连接的最长时间，单位是秒，默认为75s。
    第二个值可以省略，它是针对客户端的浏览器来设置的，可以通过curl -I看到header信息中有一项Keep-Alive: timeout=60，如果不设置就没有这一项。
    第二个数值设置后，浏览器就会根据这个数值决定何时主动关闭连接，Nginx服务器就不操心了。但有的浏览器并不认可该参数。
    
####send_timeout 

    这个超时时间是发送响应的超时时间，即Nginx服务器向客户端发送了数据包，但客户端一直没有去接收这个数据包。
    如果某个连接超过send_timeout定义的超时时间，那么Nginx将会关闭这个连接。

####client_max_body_size 10m;

    浏览器在发送含有较大HTTP包体的请求时，其头部会有一个Content-Length字段，client_max_body_size是用来限制Content-Length所示值的大小的。
    这个限制包体的配置不用等Nginx接收完所有的HTTP包体，就可以告诉用户请求过大不被接受。会返回413状态码。
    例如，用户试图上传一个1GB的文件，Nginx在收完包头后，发现Content-Length超过client_max_body_size定义的值，
    就直接发送413(Request Entity Too Large)响应给客户端。

####gzip on；

    是否开启gzip压缩。
    
####gzip_min_length 1k; 

    设置允许压缩的页面最小字节数，页面字节数从header头得content-length中进行获取。默认值是20。建议设置成大于1k的字节数，小于1k可能会越压越大。

####gzip_buffers 4 16k;

    设置系统获取几个单位的buffer用于存储gzip的压缩结果数据流。4 16k代表分配4个16k的buffer。
    
####gzip_http_version 1.1;

    用于识别 http 协议的版本，早期的浏览器不支持 Gzip 压缩，用户会看到乱码，所以为了支持前期版本加上了这个选项。
    如果你用了Nginx反向代理并期望也启用Gzip压缩的话，由于末端通信是http/1.1，故请设置为 1.1。
    
####gzip_comp_level 6; 

    gzip压缩比，1压缩比最小处理速度最快，9压缩比最大但处理速度最慢(传输快但比较消耗cpu)
    
###gzip_types mime-type ... ;

    匹配mime类型进行压缩，无论是否指定,”text/html”类型总是会被压缩的。
    在conf/mime.conf里查看对应的type。
    
    示例：gzip_types       text/plain application/x-javascript text/css text/html application/xml;
    
####gzip_proxied any;

    Nginx作为反向代理的时候启用，决定开启或者关闭后端服务器返回的结果是否压缩，匹配的前提是后端服务器必须要返回包含”Via”的 header头。
    
    以下为可用的值：
    off - 关闭所有的代理结果数据的压缩
    expired - 启用压缩，如果header头中包含 "Expires" 头信息
    no-cache - 启用压缩，如果header头中包含 "Cache-Control:no-cache" 头信息
    no-store - 启用压缩，如果header头中包含 "Cache-Control:no-store" 头信息
    private - 启用压缩，如果header头中包含 "Cache-Control:private" 头信息
    no_last_modified - 启用压缩,如果header头中不包含 "Last-Modified" 头信息
    no_etag - 启用压缩 ,如果header头中不包含 "ETag" 头信息
    auth - 启用压缩 , 如果header头中包含 "Authorization" 头信息
    any - 无条件启用压缩
    
####gzip_vary on；

    和http头有关系，会在响应头加个 Vary: Accept-Encoding ，可以让前端的缓存服务器缓存经过gzip压缩的页面，例如，用Squid缓存经过Nginx压缩的数据。
    

