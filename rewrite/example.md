####Rewrite实战
    
    本部分内容为nginx生产环境中使用的场景示例。
    
#####域名跳转（域名重定向）

    示例1（不带条件的）：
    server{
        listen 80;
        server_name www.aminglinux.com;
        rewrite /(.*) http://www.aming.com/$1 permanent;
        .......
        
    }
    
    示例2（带条件的）：
    server{
        listen 80;
        server_name www.aminglinux.com aminglinux.com;
        if ($host != 'www.aminglinux.com')
        {
            rewrite /(.*) http://www.aminglinux.com/$1 permanent;
        }
        .......
        
    }
    示例3（http跳转到https）：
    server{
        listen 80;
        server_name www.aminglinux.com;
        rewrite /(.*) https://www.aminglinux.com/$1 permanent;
        .......
        
    }
    示例4（域名访问二级目录）
    server{
        listen 80;
        server_name bbs.aminglinux.com;
        rewrite /(.*) http://www.aminglinux.com/bbs/$1 last;
        .......
        
    }
    示例5（静态请求分离）
    server{
        listen 80;
        server_name www.aminglinux.com;
        location ~* ^.+.(jpg|jpeg|gif|css|png|js)$
        {
            rewrite /(.*) http://img.aminglinux.com/$1 permanent;
        }

        .......
        
    }
    或者：
    server{
        listen 80;
        server_name www.aminglinux.com;
        if ( $uri ~* 'jpg|jpeg|gif|css|png|js$')
        {
            rewrite /(.*) http://img.aminglinux.com/$1 permanent;
        }

        .......
        
    }
    
#####防盗链

    示例6
    server{
        listen 80;
        server_name www.aminglinux.com;
        location ~* ^.+.(jpg|jpeg|gif|css|png|js|rar|zip|flv)$
        {
            valid_referers none blocked server_names *.aminglinux.com aminglinux.com *.aming.com aming.com;
            if ($invalid_referer)
            {
                rewrite /(.*) http://img.aminglinux.com/images/forbidden.png;
            }
        }

        .......
        
    }
    说明：*这里是通配，跟正则里面的*不是一个意思，none指的是referer不存在的情况（curl -e 测试），
          blocked指的是referer头部的值被防火墙或者代理服务器删除或者伪装的情况，
          该情况下，referer头部的值不以http://或者https://开头（curl -e 后面跟的referer不以http://或者https://开头）。
    或者：
        location ~* ^.+.(jpg|jpeg|gif|css|png|js|rar|zip|flv)$
        {
            valid_referers none blocked server_names *.aminglinux.com *.aming.com aminglinux.com aming.com;
            if ($invalid_referer)
            {
                return 403;
            }
        }
    

#####伪静态

    示例7(discuz伪静态)：
    location /  {
        rewrite ^([^\.]*)/topic-(.+)\.html$ $1/portal.php?mod=topic&topic=$2 last;
        rewrite ^([^\.]*)/forum-(\w+)-([0-9]+)\.html$ $1/forum.php?mod=forumdisplay&fid=$2&page=$3 last;
        rewrite ^([^\.]*)/thread-([0-9]+)-([0-9]+)-([0-9]+)\.html$ $1/forum.php?mod=viewthread&tid=$2&extra=page%3D$4&page=$3 last;
        rewrite ^([^\.]*)/group-([0-9]+)-([0-9]+)\.html$ $1/forum.php?mod=group&fid=$2&page=$3 last;
        rewrite ^([^\.]*)/space-(username|uid)-(.+)\.html$ $1/home.php?mod=space&$2=$3 last;
        rewrite ^([^\.]*)/(fid|tid)-([0-9]+)\.html$ $1/index.php?action=$2&value=$3 last;
    }
    
#####rewrite多个条件的并且

    示例8：
    location /{
        set $rule 0;
        if ($document_uri !~ '^/abc')
        {
            set $rule "${rule}1";
        }
        if ($http_user_agent ~* 'ie6|firefox')
        {
           set $rule "${rule}2";
        }
        if ($rule = "012")
        {
            rewrite /(.*) /abc/$1 redirect;
        }
    }
    