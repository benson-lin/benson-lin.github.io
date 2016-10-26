---
title: "Managing Static Files | Djingo "
layout: post
date: 2016-10-26
permalink : /post/python-django-managing-static-files
tag:
- Python
blog: true
---

# Django--static静态文件引用

首先确保settings.py中的INSTALLED_APPS中包含django.contrib.staticfiles

在`settings.py`文件中可以看到一行内容为：`STATIC_URL = '/static/'`，该路径是在APP目录下的static文件夹(与templates文件夹所在位置一致)。

如下是项目文件目录结构例子

![](img/2016-10-26-python-django-static-file-01.png)

其中 index.html 引入了 test.js 作为测试

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>{{title}}</title>
<script type="text/javascript" src="/static/js/test.js"></script>
</head>
<body>

</body>
</html>
```

当运行 `python manage.py runserver` 后，当访问 `127.0.0.1:8000`时会访问到该页面(urls中已配置，因为不是本节的重点，所以不将文件展示出来)

可以看到访问成功，响应码是200

![](img/2016-10-26-python-django-static-file-02.png)

在服务器端的日志也看到`[26/Oct/2016 17:36:13] "GET /static/js/test.js HTTP/1.1" 200 0`

如果希望修改静态文件目录，只需要修改`STATIC_URL`的值即可

Django讲述了另外两种方法，下面是部分截图。原链接在[这里](https://docs.djangoproject.com/en/1.10/howto/static-files/)

![](img/2016-10-26-python-django-static-file-04.png)

## 静态文件的一般处理方式

一般我们的静态文件是放在单独的服务器的，比如我们访问github.com，我们看到有某个css的请求路径是`https://assets-cdn.github.com/assets/frameworks-96d3c90d9d94b08c8ad5ae6b8330d0ee8a8488a340e22ff175ad5f68d1e7829f.css`，使用了CDN。

上面的情况一般用在小项目或者在调试中使用，因此我们有可能需要将静态文件收集在一起，最后放到如Apache或Nginx等静态资源服务器上。

我们在settings.py中加入`STATIC_ROOT = os.path.join(BASE_DIR, 'collected_static')`，然后运行`python manage.py collectstatic`命令，刷新文件夹发现在项目根目录下生成了一个collected_static的文件夹，以前放在app下static中的静态文件全部拷贝到这个文件夹中。最后将这些文件放到静态资源服务器即可。（`STATIC_ROOT`一般设置为绝对路径或者说部署的路径，比如`/var/www/example.com/static/`）

![](img/2016-10-26-python-django-static-file-03.png)

官网[这里](https://docs.djangoproject.com/en/1.10/howto/static-files/#deployment)也简单的描述了该功能

