####nginx 常用全局变量

| 变量       | 说明    |
| :--------   | :-----   | 
|$args       |请求中的参数，如www.123.com/1.php?a=1&b=2的$args就是a=1&b=2 |
|$content_length |HTTP请求信息里的"Content-Length" |
|$conten_type    |  HTTP请求信息里的"Content-Type"   |
|$document_root|nginx虚拟主机配置文件中的root参数对应的值|
|$document_uri|当前请求中不包含指令的URI，如www.123.com/1.php?a=1&b=2的$document_uri就是1.php,不包含后面的参数|
|$host|主机头，也就是域名|
|$http_user_agent|客户端的详细信息，也就是浏览器的标识，用curl -A可以指定|
|$http_cookie|客户端的cookie信息|
|$limit_rate|如果nginx服务器使用limit_rate配置了显示网络速率，则会显示，如果没有设置， 则显示0|
|$remote_addr|客户端的公网ip|
|$remote_port|客户端的port|
|$remote_user|如果nginx有配置认证，该变量代表客户端认证的用户名|
|$request_body_file|做反向代理时发给后端服务器的本地资源的名称|
|$request_method|请求资源的方式，GET/PUT/DELETE等|
|$request_filename|当前请求的资源文件的路径名称，相当于是$document_root/$document_uri的组合|
|$request_uri|请求的链接，包括$document_uri和$args|
|$scheme|请求的协议，如ftp,http,https|
|$server_protocol|客户端请求资源使用的协议的版本，如HTTP/1.0，HTTP/1.1，HTTP/2.0等|
|$server_addr|服务器IP地址|
|$server_name|服务器的主机名|
|$server_port|服务器的端口号|
|$uri|和$document_uri相同|
|$http_referer|客户端请求时的referer，通俗讲就是该请求是通过哪个链接跳过来的，用curl -e可以指定|
