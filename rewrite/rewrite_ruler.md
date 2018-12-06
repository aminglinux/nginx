####rewrite规则

    格式：rewrite  regex replacement [flag] 
    
    * rewrite配置可以在server、location以及if配置段内生效
    
    * regex是用于匹配URI的正则表达式，其不会匹配到$host（域名）
    
    * replacement是目标跳转的URI，可以以http://或者https://开头，也可以省略掉$host，直接写$request_uri部分（即请求的链接）
    
    * flag，用来设置rewrite对URI的处理行为，其中有break、last、rediect、permanent，其中break和last在前面已经介绍过，
    rediect和permanent的区别在于，前者为临时重定向(302)，而后者是永久重定向(301)，对于用户通过浏览器访问，这两者的效果是一致的。
    但是，对于搜索引擎蜘蛛爬虫来说就有区别了，使用301更有利于SEO。所以，建议replacemnet是以http://或者https://开头的flag使用permanent。
    
#####示例1

    location / {
        rewrite /(.*) http://www.aming.com/$1 permanent;
    }
    说明：.*为正则表达式，用()括起来，在后面的URI中可以调用它，第一次出现的()用$1调用，第二次出现的()用$2调用，以此类推。
    
#####示例2

    location / {
        rewrite /.* http://www.aming.com$request_uri permanent;
    }
    说明：在replacement中，支持变量，这里的$request_uri就是客户端请求的链接
    
#####示例3

    server{
        listen 80;
        server_name www.123.com;
        root /tmp/123.com;
        index index.html;
        rewrite /(.*) /abc/$1 redirect;
    }
    说明：本例中的rewrite规则有问题，会造连续循环，最终会失败，解决该问题有两个方案。
    关于循环次数，经测试发现，curl 会循环50次，chrome会循环80次，IE会循环120次，firefox会循环20次。

#####示例4

    server{
        listen 80;
        server_name www.123.com;
        root /tmp/123.com;
        index index.html;
        rewrite /(.*) /abc/$1 break;
    }
    说明：在rewrite中使用break，会避免循环。
    
#####示例5

    server{
        listen 80;
        server_name www.123.com;
        root /tmp/123.com;
        index index.html;
        if ($request_uri !~ '^/abc/')
        {
            rewrite /(.*) /abc/$1 redirect;
        }
    }
    说明：加一个条件限制，也可以避免产生循环