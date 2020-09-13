---
title: 使用zerotier配置moon结点优化使用体验
tags:
  - 网络应用
categories:
  - 技术分享
abbrlink: c783d70
date: 2020-01-03 17:02:16
---
# zerotier介绍
这是一款能够帮助你轻松实现虚拟局域网的软件。通过它，你可以将100台设备(免费版账号)加入自己创建的虚拟局域网。且该软件支持支持 Windows、macOS、Linux 三大桌面平台，iOS、Android 两大移动平台。[官网地址](https://www.zerotier.com/)

# 注册与使用
该过程十分简单，按照以下四步即可，具体过程网上资料很多，不再赘述。<br>

1. [下载zerotier](https://www.zerotier.com/download/)
2. [注册](https://my.zerotier.com/login)
3. 创建自己的Network
4. 将需要的设备加入虚拟局域网  


# 配置中转moon
国内用户实际使用zerotier时会发现延迟很高。比如在自己搭建的虚拟局域网中使用windows远程桌面，经常卡屏。怎么办呢？别担心，zerotier官方已经帮你想好了解决方案。  
首先，明确几个名词

- planet 行星节点，zerotier官方提供的根服务器
- moon 卫星节点 用户自己搭建的服务器，可以中转加速
- leaf 相当于用户实际想组成局域网的那些客户端



## 配置步骤
为了避免不必要的麻烦，以下所有命令都默认在管理员状态下进行

	sudo -i//该命令可进入管理员状态
### 1、拥有一台自己的vps 
巧妇难为无米之炊，没有自己的服务器何谈加速一说。moon节点服务器不需要多高的配置，关于如何购买请自行搜索。很多平台现在也支持支付宝等国内软件进行支付。如果你只有短期需求，也可以使用谷歌或亚马逊提供的一年免费云服务，有支持外币的信用卡就可以申请。
### 2、在moon服务器中安装zerotier
复制以下命令  

	curl -s https://install.zerotier.com/ | sudo bash
### 3、将moon节点加入网络
归根结底，moon节点只是你个人的一个加速器，zerotier官方提供的NetWork环境还是必不可少的。moon节点和所有的客户机都需要加入你在zerotier官网创建的network。
在moon服务中输入以下命令，其中network id是你在zerotier官网创建的network的id  

    zerotier-cli join <network id>
### 4、生成moon.json配置文件
	cd /var/lib/zerotier-one //进入目录 之后的命令基本都在该目录执行
	zerotier-idtool initmoon identity.public > moon.json //生成文件
### 5、修改配置文件
使用以下命令进入配置文件，按**i**进入编辑状态。

	[root@moon zerotier-one]# vi moon.json
修改**stableEndpoints**的值，同时记录一下**id**的值，是一个10位字符串，后面有用。

	"stableEndpoints": [ "你的vps的ip/9993" ] //9993是zerotier指定的默认端口
修改完成后，按esc退出编辑状态，然后输入**:wq**保存并退出。
### 6、生成moon文件
首先在/var/lib/zerotier-one目录新建**moons.d**文件夹  
	
	mkdir moons.d
生成文件  

	zerotier-idtool genmoon moon.json

并将生成的 moon 文件放到该文件夹中，其中xxx是之前提到的配置文件中的id

	mv 000000xxxxxxxxxx.moon moons.d/

### 7、重启vps的zerotier

	killall -9 zerotier-one
如果提示**killall: command not found**，则需要先安装  
debian、ubuntu系统下：
	
	apt-get install psmisc
centos 下：

	yum install psmisc

至此，vps上的操作完成。
### 8、将客户机连接到你的moon
zerotier在不同系统的安装目录如下

	Windows: C:\ProgramData\ZeroTier\One
	Macintosh: /Library/Application Support/ZeroTier/One (在 Terminal 中应为 /Library/Application\ Support/ZeroTier/One)
	Linux: /var/lib/zerotier-one
以**管理员**身份打开命令行，进到安装目录下。输入以下命令  
注意：如果没有管理员权限，可能会报错

	zerotier-cli orbit id id //这里的id是之前让你记录的moon.json中的10位字符串

一般到这里就已经连接成功了。如果你还是有报错，可以尝试直接将000000xxxxxxxxxx.moon文件手动拷贝到客户机的**zerotier-one/moons.d**文件夹中。关于如何将vps中的文件下载到本地，可直接使用xshell自带的文件传输功能。  
重启客户机的zerotier，以**管理员**权限进入命令行的zerotier安装目录，输入以下命令

	zerotier-cli listpeers
如果出现了带**MOON**字样的节点，代表连接成功。

	200 listpeers id 你的vps公网ip/9993;2160;2058 196 1.4.6 MOON

现在，你可以重复步骤8，将所有需要的客户机连接到你的私人moon。
Congratulations!



