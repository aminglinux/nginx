####Nginx请求流程
![image](https://coding.net/u/aminglinux/p/nginx/git/raw/master/4z/nginx_phase.png)

####Nginx各phase对应的模块
| phase      | 对应模块    |
| :--------   | :-----   | 
| NGX_HTTP_POST_READ_PHASE       | ngx_http_realip_module      |
| NGX_HTTP_SERVER_REWRITE_PHASE        | ngx_http_rewrite_module     |
|NGX_HTTP_FIND_CONFIG_PHASE|ngx_http_core_module|
|NGX_HTTP_REWRITE_PHASE |ngx_http_rewrite_module、ngx_http_core_module|
|NGX_HTTP_POST_REWRITE_PHASE|无对应模块|
|NGX_HTTP_PREACCESS_PHASE|ngx_http_limit_conn_module、ngx_http_limit_req_module|
|NGX_HTTP_ACCESS_PHASE|ngx_http_access_module|
|NGX_HTTP_POST_ACCESS_PHASE|无对应模块|
|NGX_HTTP_TRY_FILES_PHASE|ngx_http_core_module|
|NGX_HTTP_CONTENT_PHASE|ngx_http_autoindex_module、ngx_http_core_module、ngx_http_index_module、ngx_http_proxy_module、ngx_http_stub_status_module等|
|NGX_HTTP_LOG_PHASE|ngx_http_log_module|

####各phase说明
######NGX_HTTP_POST_READ_PHASE
```
nginx读取并解析完请求头之后就进入了post_read 阶段，它位于uri被重写之前，这个阶段允许nginx改变请求头中ip地址的值
```
######NGX_HTTP_SERVER_REWRITE_PHASE
```
这个阶段主要进行初始化全局变量，或者server级别的重写。如果把重写指令放到 server 中，那么就进入了server rewrite 阶段。
```

######NGX_HTTP_FIND_CONFIG_PHASE
```
这个阶段使用重写之后的uri来查找对应的location，值得注意的是该阶段可能会被执行多次，因为也可能有location级别的重写指令。
```
######NGX_HTTP_REWRITE_PHASE
```
如果把重写指令放到 location中，那么就进入了rewrite phase，这个阶段是location级别的uri重写阶段，重写指令也可能会被执行多次
```
######NGX_HTTP_POST_REWRITE_PHASE
```
location级别重写的下一阶段，用来检查上阶段是否有uri重写，并根据结果跳转到合适的阶段
```
######NGX_HTTP_PREACCESS_PHASE
```
访问权限控制的前一阶段，该阶段在权限控制阶段之前，一般也用于访问控制，比如限制访问频率，链接数等
```
######NGX_HTTP_ACCESS_PHASE
```
访问权限控制阶段，比如基于ip黑白名单的权限控制，基于用户名密码的认证控制等
```
######NGX_HTTP_POST_ACCESS_PHASE
```
访问权限控制的后一阶段，该阶段根据权限控制阶段的执行结果进行相应处理
```
######NGX_HTTP_TRY_FILES_PHASE
```
ngx_http_core_module的try_files指令的处理阶段，如果没有配置try_files指令，则该阶段被跳过。
当try_files用于server配置段时一般是初始化作用，用来加载一些文件。
```
######NGX_HTTP_CONTENT_PHASE
```
内容生成阶段，该阶段产生响应，并发送到客户端
```
######NGX_HTTP_LOG_PHASE
```
日志记录阶段，该阶段记录访问日志
```

######参考内容：
```
http://tengine.taobao.org/book/chapter_02.html#id12
https://blog.csdn.net/qinyushuang/article/details/44567885 
```