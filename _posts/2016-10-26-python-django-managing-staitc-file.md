---
title: "Managing Static Files | Django "
layout: post
date: 2016-10-26
tags:
- Python
categories: Python
excerpt: Python Django框架引用静态文件问题
---

# Django 静态文件引用

首先确保settings.py中的INSTALLED_APPS中包含django.contrib.staticfiles

在`settings.py`文件中可以看到一行内容为：`STATIC_URL = '/static/'`，该路径是在APP目录下的static文件夹(与templates文件夹所在位置一致)。

如下是项目文件目录结构例子

![](/assets/images/2016-10-26-python-django-static-file-01.png)

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

![](/assets/images/2016-10-26-python-django-static-file-02.png)

在服务器端的日志也看到`[26/Oct/2016 17:36:13] "GET /static/js/test.js HTTP/1.1" 200 0`

如果希望修改静态文件目录，只需要修改`STATIC_URL`的值即可

Django讲述了另外两种方法，下面是部分截图。原链接在[这里](https://docs.djangoproject.com/en/1.10/howto/static-files/)

![](/assets/images/2016-10-26-python-django-static-file-04.png)

## 静态文件的一般处理方式

一般我们的静态文件是放在单独的服务器的，比如我们访问github.com，我们看到有某个css的请求路径是`https://assets-cdn.github.com/assets/frameworks-96d3c90d9d94b08c8ad5ae6b8330d0ee8a8488a340e22ff175ad5f68d1e7829f.css`，使用了CDN。

而且在生产环境下一般是把静态文件放在项目根目录下的static目录下;再者，django内置环境的性能差，所以一般也将整个项目部署到apache下。

上面的情况一般用在开发或调试环境中中使用，在生产环境中我们需要将静态文件收集在一起，最后放到如Apache或Nginx等服务器上。由这些服务器来处理映射

官网原文如下：

> It’s intended only for use while developing. (We’re in the business of making Web frameworks, not Web servers.)

我们在settings.py中加入`STATIC_ROOT = os.path.join(BASE_DIR, 'collected_static')`，然后运行`python manage.py collectstatic`命令，刷新文件夹发现在项目根目录下生成了一个collected_static的文件夹，以前放在app下static中的静态文件全部拷贝到这个文件夹中。最后将这些文件放到服务器即可。（这里只是为了举例，所以放到app目录下，实际上`STATIC_ROOT`一般设置为绝对路径或者说部署的路径，比如`/var/www/example.com/static/`）

![](/assets/images/2016-10-26-python-django-static-file-03.png)

官网[这里](https://docs.djangoproject.com/en/1.10/howto/static-files/#deployment)也简单的描述了该功能



