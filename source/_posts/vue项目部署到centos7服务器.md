---
title: vue项目部署到centos7服务器
tags:
  - 部署
category:
  - linux
abbrlink: 4f35d1fd
date: 2020-04-12 20:22:32
---
## 项目build生成静态文件
首先将写好的vue项目打包
<!--more-->

    npm run build

打包后可以看到项目中多出一个*dist*文件夹，其中包含了一个*static*文件夹和index.html页面
##将dist文件上传到服务器
使用xshell连接服务器，进入你想要存放的目录，本文以/usr/share为例

    cd /usr/share
在当前目录下输入rz命令，选择压缩后的dist文件，上传

如果提示command not found，则需要安装rz命令。先切换到root用户，输入以下命令

    yum -y install lrzsz

安装后就可以正常使用了
## 服务器安装配置nginx
> Nginx是异步框架的网页服务器，也可以用作反向代理、负载平衡器和HTTP缓存。——维基百科

### 安装
因为yum中没有nginx 所以通过以下命令将nginx加入

     rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
使用yum安装
    
    yum install nginx
查看版本，验证是否安装成功

    nginx -v
启动nginx

    service nginx start
现在，在浏览器中输入服务器的公网ip，就可以看到nginx页面了
### 修改配置文件
在服务器中打开nginx配置文件，本文路径在*/etc/*下

    vim /etc/nginx/nginx.conf
打开的配置文件修改对应项

     server {
        listen       80;
        listen       [::]:80;
        server_name  你的服务器绑定的域名;
        root         你的dist文件存放路径;
        location / {
			root     你的dist文件存放路径;
			index  index.html index.htm;
        }
    }

重启nginx，以后每次修改配置文件都要重启才能生效

    service nginx restart 

现在访问你的服务器ip或域名，就能看到你的项目，恭喜！