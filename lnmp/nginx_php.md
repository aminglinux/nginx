####配置Nginx和php
```
配置如下（在server部分添加）：
    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/tmp/php-fcgi.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

配置说明：
1 fastcgi_params文件在/usr/local/nginx/conf/下面，其内容为fastcgi相关的变量
2 fastcgi_pass后面跟的是php-fpm服务监听地址，可以是IP:PORT，也可以是unix socket地址，也支持upstream的地址
3 fastcgi_index定义索引页，如果在server内其他部分有定义index参数，该配置可以忽略
4 fastcgi_param这行其实可以在fastcgi_params文件里面定义SCRIPT_FILENAME变量，这个变量如果不定义，php的请求是没办法访问的。
```
