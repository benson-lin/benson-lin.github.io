---
title: "Github Multiple SSH Keys Settings"
layout: post
date: 2016-09-08
tags:
- Other
- Git
categories: GIT
excerpt: Github 如何在一台电脑使用多个账号提交代码！
---

## 生成两个用户的ssh key

注意：看到下面需要输入key名的时候不要直接回车，其它输入可以直接回车！

```
Enter file in which to save the key (/c/Users/Zebin/.ssh/id_rsa): 111
```

**1. 生成并添加第一个ssh key**

第一次使用ssh生成key，默认会在用户~（根目录）下生成 id_rsa, id_rsa.pub 2个文件；所以需要添加多个ssh key时也会生成对应的私钥和公钥。

```linux
$ ssh-keygen -t rsa -C "1096101803@qq.com"
```

在Git Bash中执行这条命令，key名填写 1096101803_id_rsa, 会在 ~/.ssh/ 目录下生成 1096101803_id_rsa 和 1096101803_id_rsa.pub 两个文件，用文本编辑器将 id_rsa_pub 中的内容复制一下粘贴到github(gitlab)上。

**2. 生成并添加第二个ssh key**

```linux
$ ssh-keygen -t rsa -C "2205763695@qq.com"
```

注意不要一路回车，要给这个文件起一个名字， 比如叫 2205763695, 会在 ~/.ssh/ 目录下生成 2205763695_id_rsa 和 2205763695_id_rsa.pub 两个文件

![](/assets/images/2016-09-08-username-id-rsa.png)

然后分别打开pub文件拷贝到对应GitHub后台。这个步骤流程就不详说了。

## 修改配置文件

在 ~/.ssh 目录下新建一个config文件

添加内容：

```text
# github1
Host benson-lin
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/1096101803_id_rsa

# github2
Host benson-lin-psychology
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/2205763695_id_rsa
```

Host后面是github别名，新建的帐号使用这个别名做克隆和更新。
其规则就是：从上至下读取config的内容，在每个Host下寻找对应的私钥。

这里将GitHub SSH仓库地址中的git@github.com替换成新建的Host别名如：benson-lin，那么原地址 是：git@github.com:username/Mywork.git，替换后应该是：benson-lin:username/Mywork.git.




## 测试


```linux
$ ssh -T git@benson-lin-psychology
Hi benson-lin-psychology! You've successfully authenticated, but GitHub does not provide shell access.
 $ ssh -T git@benson-lin
Hi benson-lin! You've successfully authenticated, but GitHub does not provide shell access.

而不是
$ ssh -T git@github.com
```

注意：Hi后面是真正返回的github账户的用户名，而ssh命令中输入的是别名，只是这里恰好相同而已

![](/assets/images/2016-09-08-result.png)


## 应用 

测试成功，尝试克隆ruanyf账号下的远程仓库git-demo。分两种情况如下：

**1、本地已经创建或已经clone到本地：**

```linux
$ git remote rm origin
$ git remote add origin git@benson-lin:ruanyf/git-demo.git
```

**2、clone仓库时对应配置host对应的账户**

```
git clone git@benson-lin:ruanyf/git-demo.git
```

这样这个仓库就属于benson-lin这个别名账户的了，