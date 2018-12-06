####Nginx访问日志过滤
```
一个网站，会包含很多元素，尤其是有大量的图片、js、css等静态元素。
这样的请求其实可以不用记录日志。
```
#####配置示例
```
location ~* ^.+\.(gif|jpg|png|css|js)$ 
{
    access_log off;
}

或
location ~* ^.+\.(gif|jpg|png|css|js)$                                      
{
    access_log /dev/null;
}
```
