user nobody; # 定义运行nginx服务的用户,还可以加上组,如 user nobody nobody

worker_processes  8; #开启8个工作进程
worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000; #将8个工作进程固定在8个cpu上

#以下为4核CPU的范例
#worker_processes  4;
#worker_cpu_affinity 0001 0010 0100 1000;

error_log logs/error.log crit; #定义错误日志的路径和级别。错误日志级别：常见的错误日志级别有[debug|info|notice|warn|error|crit|alert|emerg]，级别越高记录的信息越少。

pid logs/nginx.pid; #定义nginx进程pid文件所在路径，可以是相对路径，也可以是绝对路径。

worker_rlimit_nofile 1024000; #定义nginx最多打开文件数限制。如果没设置的话，这个值为操作系统（ulimit -n）的限制保持一致。

events {

    use epoll; #Nginx服务器提供了多个事件驱动器模型来处理网络消息。epol这种方式是在Linux 2.6+内核中最高效的方式
    worker_connections  65535; #定义每个work_process同时开启的最大连接数，即允许最多只能有这么多连接。
    accept_mutex on; #当某一个时刻只有一个网络连接请求服务器时，服务器上有多个睡眠的进程会被同时叫醒，这样会损耗一定的服务器性能。Nginx中的accept_mutex设置为on，将会对多个Nginx进程（worker processer）接收连接时进行序列化，防止多个进程争抢资源。
    multi_accept on; #nginx worker processer可以做到同时接收多个新到达的网络连接，前提是把该参数设置为on。默认为off，即每个worker process一次只能接收一个新到达的网络连接。
} 

http {
    include        /etc/nginx/mime.types; # 定义nginx能识别的网络资源媒体类型（如，文本、html、js、css、流媒体等
    default_type  application/octet-stream; #定义默认的type，如果不定义该项，默认为text/plain.
    client_max_body_size 1024M; #定义允许最大可以上传多大的文件，超过该值就会报413
    log_format main  '$remote_addr $http_x_forwarded_for [$time_local]'
    '$host "$request_uri" $status'
    '"$http_referer" "$http_user_agent"';
    #这里定义日志的格式，其中main为日志格式的名字，后面的为nginx的内部变量组成的一串字符串。
    sendfile        on; #使用内核的FD文件传输功能，可以减少user mode和kernel mode的切换，从而提升服务器性能。
    sendfile_max_chunk 128k; #该参数限定Nginx worker process每次调用sendfile()函数传输数据的最大值，默认值为0，如果设置为0则无限制。
    tcp_nopush      on; #设置为on时，会调用tcp_cork方法进行数据传输。当应用程序产生数据时，内核不会立马封装包，而是当数据量积累到一定量时才会封装，然后传输。
    tcp_nodelay     on; #不缓存data-sends（关闭 Nagle 算法），这个能够提高高频发送小数据报文的实时性。
    server_tokens   off;#将Nginx版本信息关闭，提升安全性。
    keepalive_timeout 65 60; #该参数有两个值，第一个值设置nginx服务器与客户端会话结束后仍旧保持连接的最长时间，单位是秒，默认为75s。第二个值可以省略，它是针对客户端的浏览器来设置的，可以通过curl -I看到header信息中有一项Keep-Alive: timeout=60，如果不设置就没有这一项。第二个数值设置后，浏览器就会根据这个数值决定何时主动关闭连接，Nginx服务器就不操心了。但有的浏览器并不认可该参数。
    send_timeout 10; #这个超时时间是发送响应的超时时间，即Nginx服务器向客户端发送了数据包，但客户端一直没有去接收这个数据包。如果某个连接超过send_timeout定义的超时时间，那么Nginx将会关闭这个连接。
    client_header_timeout 15; #客户端如果在该指定时间内没有加载完头部数据，则断开连接，单位是秒，默认60，可以设置为15。
    client_body_timeout 120;  #客户端如果在该指定时间内没有加载完body数据，则断开连接，单位是秒，默认60，建议大一点，因为有时候下载大文件时间会比较久。
    client_body_buffer_size 128k; #当客户端以POST方法提交一些数据到服务端时，会先写入到client_body_buffer中，如果buffer写满会写到临时文件里。
    client_header_buffer_size 4k; #客户端请求头部的缓冲区大小，这个可以根据你的系统分页大小来设置，一般一个请求头的大小不会超过1k，不过由于一般系统分页都要大于1k,所以还是要大一些。
    large_client_header_buffers 4 8k; #对于比较大的header（超过client_header_buffer_size）将会使用该部分buffer，两个数值，第一个是个数，第二个是每个buffer的大小。
    open_file_cache max=204800 inactive=20s; #max设定缓存文件的数量，inactive设定经过多长时间文件没被请求后删除缓存。
    open_file_cache_min_uses 1; #open_file_cache指令中的inactive参数时间内文件的最少使用次数，如,将该参数设置为1，则表示，如果文件在inactive时间内一次都没被使用，它将被移除。
    open_file_cache_valid 30s; #指多长时间检查一次缓存的有效信息。建议设置为30s。

    gzip on;  #开启gzip功能
    gzip_min_length 1024;  #设置请求资源超过该数值才进行压缩，单位字节
    gzip_buffers 16 8k; #设置压缩使用的buffer大小，第一个数字为数量，第二个为每个buffer的大小
    gzip_comp_level 6; #设置压缩级别，范围1-9,9压缩级别最高，也最耗费CPU资源
    gzip_types text/plain text/xhtml text/css text/js text/csv application/javascript application/x-javascript application/json application/xml text/xml application/atom+xml application/rss+xml application/vnd.android.package-archive application/vnd.iphone; #指定哪些类型的文件需要压缩
    gzip_disable "MSIE 6\."; #IE6浏览器不启用压缩
    gzip_proxied any; #Nginx作为反向代理的时候启用，决定开启或者关闭后端服务器返回的结果是否压缩，匹配的前提是后端服务器必须要返回包含”Via”的 header头。
    
    include conf.d/*.conf; #要加载conf.d/下的所有.conf配置文件
}


##以下为conf.d/example.conf的内容
server {

    listen      443;  #监听端口为443，可以自定义其他端口，也可以加上IP地址，如，listen 127.0.0.1:8080;
    server_name  aaa.com aaa.net; #定义网站域名，可以写多个，用空格分隔。
    #以下配置为开启ssl认证
    ssl_certificate sslkey/www.crt; #指定crt文件路径
    ssl_certificate_key sslkey/www.key; #指定私钥文件路径
    ssl_session_cache   shared:SSL:10m; #设置ssl会话使用的缓存为10M
    ssl_session_timeout 10m; #ssl会话超时时间为10分钟
    ssl_protocols TLSv1.1 TLSv1.2; #ssl版本，不要加1.0了，1.0不安全
    ssl_ciphers AES256-SHA256:AES128-SHA256:AES256-SHA:AES128-SHA:!aNULL:!eNULL:!MD5:!DH:!ECDH:!DHE:!ECDHE:!RC4:!EXPORT; #开启或者关闭指定算法,这些是安全漏洞工具建议的配置
    ssl_prefer_server_ciphers on; #如果不指定默认为off，当为on时，服务器加密算法将优于客户端加密算法。

    access_log  logs/host.access.log  main; #定义访问日志，可以针对每一个server（即每一个站点）设置它们自己的访问日志。

    ##在server{}里有很多location配置段
    location / {
        root   html;  #定义网站根目录，目录可以是相对路径也可以是绝对路径。
        index  index.html index.htm; #定义站点的默认页。
    }

    error_page  404              /404.html;  #定义404页面
    error_page   500 502 503 504  /50x.html;  #当状态码为500、502、503、504时，则访问50x.html
    location = /50x.html {
        root   html;  #定义50x.html所在路径
    }

    #定义哪些目录下的php不能访问，一般要限制所有可写的目录下的php
    location ~ .*(data|config|template|attachments|forumdata|attachment|images|log|conf|cache)/.*\.php$ {
        deny all;
    }

    #设置防盗链，不记录日志，缓存过期时间为7天
    location ~* ^.+\.(swf|jpg|gif|bmp|png|jpeg|zip|rar)$ {
         expires 7d;
         access_log /dev/null;
         valid_referers none blocked server_names *.aaa.net *.aaa.com;
         if ($invalid_referer) {
              return 403;
         }
    }
    #js、css不记录日志，缓存过期时间为7天
    location ~ .*\.(js|css)$
    {
            expires      7d;
            access_log off;
    }

    #针对php的配置
    location ~ \.php$ {
            #fastcgi_pass   unix:/tmp/55188_71-fpm.sock;
            fastcgi_pass   127.0.0.1:9005; #这个是php-fpm的监听端口，nginx会把php的请求转发给php-fpm处理
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
            fastcgi_param  X-Real-IP $remote_addr;
            include        fastcgi_params;
        }

    #针对admin.php做限制
    location  /admin.php 
    {
	allow 12.13.12.12; #允许的ip
        allow 22.22.22.0/24; #允许的ip段
        deny all;
    }

}

## 反向代理示例
server {
    listen 80;
    server_name bbb.com;
    proxy_buffering on; #该参数设置是否开启proxy的buffer功能, 如果这个设置为off，那么proxy_buffers和proxy_busy_buffers_size这两个指令将会失效
    proxy_buffer_size 4k; #该参数用来设置header缓存的大小，不能低于4k
    proxy_buffers 8 4k; #这个参数设置存储被代理服务器上的数据所占用的buffer的个数和每个buffer的大小。
    proxy_busy_buffers_size 16k; #在所有的buffer里，我们需要规定一部分buffer把自己存的数据传给A，这部分buffer就叫做busy_buffer，该参数用来设置处于busy状态的buffer有多大。
    proxy_temp_path /tmp/nginx_proxy_tmp 1 2; #定义proxy的临时文件存在目录以及目录的层级，1表示层级1的目录名为一个数字(0-9),2表示层级2目录名为2个数字(00-99)
    proxy_max_temp_file_size 100M; #设置临时文件的总大小
    proxy_temp_file_write_size 16k; #设置同时写入临时文件的数据量的总大小
    
    location /
    {
        proxy_pass http://123.23.13.11/; #后端服务器的ip地址
        proxy_set_header Host   $host; #访问后端服务器时，用哪个域名访问呢，这里的$host就是server_name。
        proxy_set_header X-Real-IP      $remote_addr; # 用来设置被代理端接收到的远程客户端IP，如果不设置，则header信息中并不会透传远程真实客户端的IP地址。
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; #同上
    }
} 

