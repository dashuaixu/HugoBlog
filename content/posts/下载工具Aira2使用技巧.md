+++
title = '下载工具Aira2使用技巧'
date = 2024-02-18T22:49:04+08:00
draft = false
description = "Aira2强大的下载工具" 
tags = [ "BT", "工具",] 
categories = [ "工具"]
+++

# 批量下载文件的强大工具--Aria2

Aria2是一个强大的下载工具，我的每个电脑都必装的软件。它让下载文件变的非常简单，我曾用它下载过千万数量级文件，这是迅雷等做不到的。在下载单个大文件的时候也会使用多线程加速，良心推荐。
网上也有很多将某度网盘的下载链接转成aria2下载的教程，突破网速限制，大家可以技术上折腾一下:satisfied:

[官网](https://aria2.github.io/)

[官方文档](https://aria2.github.io/manual/en/html/index.html)

[TOC]

这里结合官方文档搬一些常用命令：体验一下强大和便捷。

Download from WEB: 下载网络资源

```
$ aria2c http://example.org/mylinux.iso
```

Download from 2 sources: 从多个源下载

```
$ aria2c http://a/f.iso ftp://b/f.iso
```

Download using 2 connections per host: 建立多个连接

```
$ aria2c -x2 http://a/f.iso
```

BitTorrent: 下载种子

```
$ aria2c http://example.org/mylinux.torrent
```

BitTorrent Magnet URI: 下载磁力链接

```
$ aria2c 'magnet:?xt=urn:btih:248D0A1CD08284299DE78D5C1ED359BB46717D8C'
```

Metalink: 下载Metalink

```
$ aria2c http://example.org/mylinux.metalink
```

Download URIs found in text file: 从文件批量下载

```
$ aria2c -i uris.txt
```

------

## 第一：安装

CentOS：

```shell
# 安装epel源
yum install epel-release
# 安装Aria2
yum -y install aria2
```

其他平台可以使用安装包：记得安装完了加入环境变量。

[各个平台安装包下载地址](https://github.com/aria2/aria2/releases/tag/release-1.35.0) 

------

## 第二：配置文件（可选）

将配置文件随便放入一个文件夹，使用的时候用绝对路径指定就好，这里以`/etc/aria2/aria2.conf`为例

一般自己使用可以不用密码验证

```shell
#用户名
#rpc-user=user
#密码
#rpc-passwd=passwd
#上面的认证方式不建议使用,建议使用下面的token方式
#设置加密的密钥
#rpc-secret=token
#允许rpc
enable-rpc=false
#允许所有来源, web界面跨域权限需要
rpc-allow-origin-all=true
#允许外部访问，false的话只监听本地端口
rpc-listen-all=true
#RPC端口, 仅当默认端口被占用时修改
rpc-listen-port=6800
#最大同时下载数(任务数), 路由建议值: 3
max-concurrent-downloads=10
#断点续传
continue=true
#同服务器连接数
max-connection-per-server=10
#最小文件分片大小, 下载线程数上限取决于能分出多少片, 对于小文件重要
min-split-size=1M
#单文件最大线程数, 路由建议值: 5
split=5
#下载速度限制
max-overall-download-limit=0
#单文件速度限制
max-download-limit=0
#上传速度限制
max-overall-upload-limit=0
#单文件速度限制
max-upload-limit=0
#断开速度过慢的连接
#lowest-speed-limit=0
#验证用，需要1.16.1之后的release版本
#referer=*
#文件保存路径, 默认为当前启动位置
dir=/root/download_imgs/imgs
#文件缓存, 使用内置的文件缓存, 如果你不相信Linux内核文件缓存和磁盘内置缓存时使用
#disk-cache=0
#另一种Linux文件缓存方式
#enable-mmap=true
#文件预分配, 能有效降低文件碎片, 提高磁盘性能. 缺点是预分配时间较长
file-allocation=prealloc
```

------

## 第三：命令行使用

下载单个文件示例：

`--conf-path=file_path` 用来指定配置文件

`-c` 断点续传

`-o` 可重命名文件

然后跟上地址即可，文件会下载到配置文件指定的目录下

```shell
[root@VM_0_48_centos download_imgs]# aria2c --conf-path=/etc/aria2/aria2.conf -c http://upload/201309/5226c647341c6.png

02/01 11:50:04 [NOTICE] Download complete: /root/download_imgs/imgs/5226c647341c6.png

Download Results:
gid   |stat|avg speed  |path/URI
======+====+===========+=======================================================
462c3b|OK  |        n/a|/root/download_imgs/imgs/5226c647341c6.png

Status Legend:
(OK):download completed.
```

------

## 第四：批量下载

接触这个工具就是为了解决快速批量下载大量文件的问题。

Aria2提供了URL列表文件输入的方式进行批量下载：

命令示例：

`aria2c --conf-path=/etc/aria2/aria2.conf -c -i ./all_imgs.txt`

-i 参数即 `--input-file` 用来指定URL文件路径

<font color="ff0000">URL列表文件格式：</font> 文件每一行为一个url，且不能使用引号

```text
http://upload/201309/5226c647341c6.png
http://upload/201309/5226c64756065.png
http://upload/201309/5226c6477aa08.png
```

<font color="ff0000">批量下载并重命名：</font> 

下载的文件名默认是链接中的文件名，要想自定义，只需要给每个文件加上`out`参数。

在文件中定义文件的名字， 在url的下一行加入out参数，且out参数的那一行必须以一个或多个**空格开头**，一般使用一个制表符tab；

```text
http://upload/201309/5226c647341c6.png
	out=aHR0cDovL3BpY2Zs.png
http://upload/201309/5226c64756065.png
	out=aHR0cDovL3BpY2Zsb3cu.png
http://upload/201309/5226c6477aa08.png
	out=aHR0cDovL3BpY2Zsb.png
```

------

## 补充

### 解决Aria2大批量文件下载，内存占用过高（爆内存）的问题

我尝试在一个txt文件中放入100万个链接进行下载，发现几分钟4G内存的服务器就爆掉了，进程被kill掉。

后来查资料发现是使用 -i 参数加载文件，默认会将文件全部一次性读取并创建连接，几百万的连接服务器肯定吃不消，Aria2提供了`--deferred-input=true|false` 来解决这个问题，默认是false，即一次性加载全部url，改为true以后即可分批读取，解决内存占用问题。如果文件没那么大，默认就好了。

改了参数以后进程内存占用25M左右，满带宽下载。

------

### 设置代理

[参考](<https://www.cnblogs.com/RhinoC/p/aria2.html>)

给**所有的请求**加代理:

使用`all-proxy="http://ip:port"`。或者写`--all-proxy=`。

```shell
aria2c --conf-path=/etc/aria2/aria2.conf all-proxy="http://47.101.212.xx:3111" http://SYS201905310727302116126929_ST.001.png
```

给**http请求**加代理:

使用`http-proxy="http://ip:port"`。

添加需要**验证**的代理：

使用`http-proxy='http://ip:port' --http-proxy-user='username' --http-proxy-passwd='password'`

或者`http-proxy='http://username:password@proxy:8080'`

<font color="ff0000">注意：</font> 用户名和密码需要是 percent-encoded 格式。比如，如果用户名是 myid@domain, 那么 percent-encoded 格式就是 myid%40domain 。
