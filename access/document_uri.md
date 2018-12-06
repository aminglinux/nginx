####Nginx基于$document_uri的访问控制
```
这就用到了变量$document_uri，根据前面所学内容，该变量等价于$uri，其实也等价于location匹配。
```

#####示例1
```
if ($document_uri ~ "/admin/")
{
    return 403;
}

说明：当请求的uri中包含/admin/时，直接返回403.

if结构中不支持使用allow和deny。

测试链接：
1. www.aminglinux.com/123/admin/1.html 匹配
2. www.aminglinux.com/admin123/1.html  不匹配
3. www.aminglinux.com/admin.php  不匹配
```

#####示例2
```
if ($document_uri = /admin.php)
{
    return 403;
}

说明：请求的uri为/admin.php时返回403状态码。

测试链接：
1. www.aminglinux.com/admin.php 匹配
2. www.aminglinux.com/123/admin.php  不匹配
```

#####示例3
```
if ($document_uri ~ '/data/|/cache/.*\.php$')
{
    return 403;
}

说明：请求的uri包含data或者cache目录，并且是php时，返回403状态码。

测试链接：
1. www.aminglinux.com/data/123.php  匹配
2. www.aminglinux.com/cache1/123.php 不匹配
```