###nginx.conf server部分配置

    server{} 包含在http{}内部，每一个server{}都是一个虚拟主机（站点）。

####以下为nginx.conf配置文件中server{}部分的内容。

        server {
        listen       80;  //监听端口为80，可以自定义其他端口，也可以加上IP地址，如，listen 127.0.0.1:8080;
        server_name  localhost; //定义网站域名，可以写多个，用空格分隔。
        #charset koi8-r; //定义网站的字符集，一般不设置，而是在网页代码中设置。
        #access_log  logs/host.access.log  main; //定义访问日志，可以针对每一个server（即每一个站点）设置它们自己的访问日志。

        ##在server{}里有很多location配置段
        location / {
            root   html;  //定义网站根目录，目录可以是相对路径也可以是绝对路径。
            index  index.html index.htm; //定义站点的默认页。
        }

        #error_page  404              /404.html;  //定义404页面

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;  //当状态码为500、502、503、504时，则访问50x.html
        location = /50x.html {
            root   html;  //定义50x.html所在路径
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #定义访问php脚本时，将会执行本location{}部分指令
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;  //proxy_pass后面指定要访问的url链接，用proxy_pass实现代理。
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;  //定义FastCGI服务器监听端口与地址，支持两种形式，1 IP:Port， 2 unix:/path/to/sockt
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;  //定义SCRIPT_FILENAME变量，后面的路径/scripts为上面的root指定的目录
        #    include        fastcgi_params; //引用prefix/conf/fastcgi_params文件，该文件定义了fastcgi相关的变量
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        # 
        #location ~ /\.ht {   //访问的url中，以/.ht开头的，如，www.example.com/.htaccess，会被拒绝，返回403状态码。
        #    deny  all;  //这里的all指的是所有的请求。
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;  //监听8000端口
    #    listen       somename:8080;  //指定ip:port
    #    server_name  somename  alias  another.alias;  //指定多个server_name

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;  //监听443端口，即ssl
    #    server_name  localhost;

    ### 以下为ssl相关配置
    #    ssl_certificate      cert.pem;    //指定pem文件路径
    #    ssl_certificate_key  cert.key;  //指定key文件路径

    #    ssl_session_cache    shared:SSL:1m;  //指定session cache大小
    #    ssl_session_timeout  5m;  //指定session超时时间
    #    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;   //指定ssl协议
    #    ssl_ciphers  HIGH:!aNULL:!MD5;  //指定ssl算法
    #    ssl_prefer_server_ciphers  on;  //优先采取服务器算法
    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}
