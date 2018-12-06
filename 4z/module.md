###Nginx模块化结构

    Nginx涉及到的模块分为核心模块、标准HTTP模块、可选HTTP模块、邮件服务模块以及第三方模块等五大类。

![image](https://coding.net/u/aminglinux/p/nginx/git/raw/master/4z/nginx_module.jpg)

####核心模块
    
    核心模块是指Nginx服务器正常运行时必不可少的模块，它们提供了Nginx最基本最核心的服务，如进程管理、权限控制、错误日志记录等。
    主要包含对两类功能的支持，一类是主体功能，包括进程管理、权限控制、错误日志记录、配置解析等，
    另一类是用于响应请求事件必需的功能，包括事件驱动机制、正则表达式解析等。
    
    ngx_core_module
    ngx_errlog_module
    ngx_conf_module
    ngx_regex_module
    ngx_events_module
    ngx_event_core_module
    ngx_epoll_module
    
    
####标准HTTP模块

    标准HTTP模块是编译Nginx后包含的模块，其支持Nginx服务器的标准HTTP功能。
    
| 模块       | 功能    |
| :--------   | :-----   | 
| ngx_http_core        | 配置端口，URI分析，服务器响应错误处理，别名控制以及其他HTTP核心事务      |
| ngx_http_access_module        | 基于IP地址的访问控制（允许/拒绝）     |
| ngx_http_auth_basic_module        | 基于HTTP的身份认证      |
|ngx_http_autoindex_module|	处理以“/”结尾的请求并自动生成目录列表|
|ngx_http_browser_module|解析HTTP请求头中的“User-Agent”域的值|
|ngx_http_charset_module|指定网页编码|
|ngx_http_empty_gif_module|从内存创建一个1 x 1的透明gif图片，可以快速调用|
|ngx_http_fastcgi_module|对FastCGI的支持|
|ngx_http_geo_module|将客户端的IP转化为键值对变量，该模块主要用来针对客户的的IP来定义变量|
|ngx_http_gzip_module|	压缩请求响应，可以减少数据传输|
|ngx_http_headers_filter_module|设置HTTP响应头|
|ngx_http_index_module|处理以“/”结尾的请求，如果没有找到该目录下的index页，就将请求转给ngx_http_autoindex_module模块处理|
|ngx_http_limit_req_module|限制来自客户端的请求的响应和处理速率|
|ngx_http_limit_conn_module|限制来自客户端的连接的响应和处理速率|
|ngx_http_log_module|自定义access日志|
|ngx_http_map_module|创建任意键值对变量|
|ngx_http_memcached_module|对Memcached的支持|
|ngx_http_proxy_module|	支持代理事务|
|ngx_http_referer_module|对HTTP头中的"referer"进行过滤处理，比如，实现防盗链功能|
|ngx_http_rewrite_module|实现nginx的rewrite功能|
|ngx_http_scgi_module|对SCGI的支持|
|ngx_http_upstream_module|定义一组服务器，可以接收来自代理、Fastcgi、Memcached的中重定向，主要用于负载均衡|

    
####可选HTTP模块

    可选HTTP模块主要用于扩展标准的HTTP功能，使其能够处理一些特殊的HTTP请求。在编译Nginx时，如果不指定这些模块，默认是不会安装的。

| 模块       | 功能    |
| :--------   | :-----   | 
|ngx_http_addition_module|在响应请求的页面开始或者结尾添加文本信息|
|ngx_http_degradation_module|在低内存的情形下允许Nginx服务器返回444错误或204错误|
|ngx_http_perl_module|在Nginx的配置文件中可以使用Perl脚本|
|ngx_http_flv_module|支持将Flash多媒体信息按照流文件传输，可以根据客户端指定的开始位置返回Flash|
|ngx_http_geoip_module|支持解析基于GeoIP数据库的客户端请求|
|ngx_google_perflools_module|支持Google Performance Tools的一套用于C++Profile的工具集|
|ngx_http_image_filter_module|支持将H.264/AAC编码的多媒体信息（后缀名通常为mp4、m4v或m4a）按照流文件传输，常与ngx_http_flv_module模块一起使用|
|ngx_http_random_index_module|Nginx接收到以“/”结尾的请求时，在对应的目录下随机选择一个文件作为index文件|
|ngx_http_secure_link_module|支持对请求链接的有效性检查|
|ngx_http_ssl_module|对HTTPS/SSL支持|
|ngx_http_stub_status_module|支持返回Nginx服务器的统计信息，一般包括处理连接的数量、连接成功的数量、处理的请求数、读取和返回的Header信息数等信息|
|ngx_http_sub_module|使用指定的字符串替换响应信息中的信息|
|ngx_http_dav_module|支持HTTP协议和WebDAV协议中PUT、DELETE、MKCOL、COPY和MOVE方法|
|ngx_http_xslt_module|将XML响应信息使用XSLT（拓展样式表转换语言）进行转换|

    
####邮件服务模块

    主要用于支持Ningx的邮件服务。
    
####第三方模块

    并非有Nginx官方提供，而是由第三方机构或者个人开发的模块，用于实现某种特殊功能。
    echo-nginx-module 支持在Nginx配置文件中使用echo、sleep、time以及exec等类shell命令
    lua-nginx-module 使Nginx支持lua脚本语言
    

    
