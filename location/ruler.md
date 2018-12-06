####nginx的location配置

    nginx location语法规则：location [=|~|~*|^~] /uri/ { … }
    nginx的location匹配的变量是$uri

| 符号       | 说明    |
| :--------   | :-----   | 
| = |   表示精确匹配|
| ^~|  表示uri以指定字符或字符串开头|
| ~ |  表示区分大小写的正则匹配|
| ~*|  表示不区分大小写的正则匹配|
|/|  通用匹配，任何请求都会匹配到|

#####规则优先级

    =  高于  ^~  高于  ~* 等于 ~  高于  /
    
#####规则示例

    location = "/12.jpg" { ... }
    如：
    www.aminglinux.com/12.jpg 匹配
    www.aminglinux.com/abc/12.jpg 不匹配
    
    location ^~ "/abc/" { ... }
    如：
    www.aminglinux.com/abc/123.html 匹配
    www.aminglinux.com/a/abc/123.jpg 不匹配
    
    location ~ "png" { ... }
    如：
    www.aminglinux.com/aaa/bbb/ccc/123.png 匹配
    www.aminglinux.com/aaa/png/123.html 匹配
    
    location ~* "png" { ... }
    如：
    www.aminglinux.com/aaa/bbb/ccc/123.PNG 匹配
    www.aminglinux.com/aaa/png/123.html 匹配
    
    
    location /admin/ { ... }
    如：
    www.aminglinux.com/admin/aaa/1.php 匹配
    www.aminglinux.com/123/admin/1.php 不匹配
    
#####小常识

    有些资料上介绍location支持不匹配 !~，
    如： location !~ 'png'{ ... }
    这是错误的，location不支持 !~
    
    如果有这样的需求，可以通过if来实现，
    如： if ($uri !~ 'png') { ... }
    
    注意：location优先级小于if
    
