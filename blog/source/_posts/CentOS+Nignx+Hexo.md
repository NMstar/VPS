---
title: CentOS + Nginx + Git + Hexo 建立自己的博客
date: 2018/4/20
tags: [CentOS, Nginx, Git, Hexo]
---

## vultr.com 购买服务器

## Git 安装配置
安装Git
``` bash
$ yum install git -y
```

从GitHub博客的配置 (这是我个人的博客地址)
```
$ git clone https://github.com/NMstar/VPS.git
```

## Hexo
安装NodeJS

``` bash
$ curl --silent --location https://rpm.nodesource.com/setup_8.x | sudo bash -
$ yum -y install nodejs
```

下载Xeon (和blog同目录)
``` bash 
$ npm install xeon
```

编译博客网站，可以看见出现了public文件夹
``` bash
$ cd /root/VPS/blog
$ xeon generate
```

## Nginx 安装和配置

安装Nginx
``` bash
$ yum install nginx -y
```

启动Nginx
``` bash
$ nginx
```
Nginx配置
``` bash
$ cd /etc/nginx/conf.d
$ vim default.conf
    改变location的目录
    location / {
        /root/VPS/blog/public
    }

$ vim /etc/nginx/nginx.conf
    改变user的权限
    user root;
```

## 自动更新脚本
auto_deploy.sh
``` bash
$ cd /root/VPS/
$ git pull
$ cd /root/VPS/blog
$ hexo generate
```

设定定时任务
``` bash
crontab -e

*/1 * * * * /bin/bash /root/VPS/blog/auto_deploy.sh
```

## 更新主题为next
> https://github.com/iissnan/hexo-theme-next

Git命令
``` bash
$ git clone --branch v5.1.2 https://github.com/iissnan/hexo-theme-next themes/next
```