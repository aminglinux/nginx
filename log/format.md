####Nginx访问日志格式
```
Nginx访问日志可以设置自定义的格式，来满足特定的需求。
```

#####访问日志格式示例
```
示例1
    log_format combined_realip '$remote_addr $http_x_forwarded_for [$time_local]'
    '$host "$request_uri" $status'
    '"$http_referer" "$http_user_agent"';

示例2
    log_format main '$remote_addr [$time_local] '
    '$host "$request_uri" $status "$request"'
    '"$http_referer" "$http_user_agent" "$request_time"';

若不配置log_format或者不在access_log配置中指定log_format，则默认格式为：
    '$remote_addr - $remote_user [$time_local] "$request" '
    '$status $body_bytes_sent "$http_referer" '
    '"$http_user_agent";

```
#####常见变量
| 变量       | 说明    |
| :--------   | :-----   | 
|$time_local|  通用日志格式下的本地时间；（服务器时间）|
|$remote_addr|客户端（用户）IP地址|
|$status| 请求状态码，如200，404，301，302等|
|$body_bytes_sent |发送给客户端的字节数，不包括响应头的大小|
|$bytes_sent|发送给客户端的总字节数|
|$request_length|请求的长度（包括请求行，请求头和请求正文）|
|$request_time |请求处理时间，单位为秒，小数的形式|
|$upstream_addr|集群轮询地址|
|$upstream_response_time|指从Nginx向后端（php-cgi)建立连接开始到接受完数据然后关闭连接为止的时间|
|$remote_user|用来记录客户端用户名称|
|$request |请求方式（GET或者POST等）+URL（包含$request_method,$host,$request_uri）|
|$http_user_agent|用户浏览器标识|
|$http_host |请求的url地址（目标url地址）的host|
|$host|等同于$http_host|
|$http_referer|来源页面，即从哪个页面转到本页，如果直接在浏览器输入网址来访问，则referer为空|
|$uri |请求中的当前URI(不带请求参数，参数位于$args)，不同于浏览器传递的$request_uri的值，它可以通过内部重定向，或者使用index指令进行修改。|
|$document_uri|等同于$uri|
|$request_uri|比$uri多了参数，即$uri+$args|
|$http_x_forwarded_for|如果使用了代理，这个参数会记录代理服务器的ip和客户端的ip|


