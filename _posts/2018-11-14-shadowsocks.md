---
layout:     post
title:      "ubuntu 配置shadowsocks 实现翻墙"
subtitle:   "shadowsocks"
date:       2018-11-14 20:00:00
author:     "憧憬"
header-img: "img/home-bg-o.jpg"
header-mask: 0.3
catalog:    true
tags:
  - 网络
  - linux
---
#### shadowsocks配置翻墙

1. 需要一台海外服务器（我用的是ubuntu18.04）
> 去将你本机的请求发送给服务器 服务器去请求一些谷歌等等的国外网站,实际上也只是做了一个代理

2. 修改一些ubuntu的基本东西
> 我在腾讯云买的是硅谷的一台按量付费的服务器，进去的时候发现dns和源都不能使用，随即去配置了 
> 还有就是要注意,买服务器的时候一定要开发你需要的端口，否则你会喷自己的
> vim /etc/resolv.conf  DNS配置
> 我的源是腾讯云的默认源,发现安装不了东西，随即我便去修改成了阿里云
> sudo cp /etc/apt/sources.list /etc/apt/sources.list.backup 备份原始文件
> vi /etc/apt/sources.list 修改文件并添加国内源


```
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

```

然后update->install shadowsocks  默认ubuntu16.04以上好像就可以直接安装 
没有的可以先去安装
sudo apt-get install python-pip
sudo apt-get install m2crypto
pip install shadowsocks   利用pip去安装shadowsocks

3. 修改shadowsocks配置文件

> 安装了shadowsocks 配置文件在 /etc/shadowsocks/config.json

文件内容如下


```
{
    "server":"0.0.0.0",                  // 所有ip都可以访问
    "server_port":8388,                  // 开放端口
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"1234567890",              // 密码
    "timeout":300,
    "method":"aes-256-cfb",             // 加密
    "fast_open": false,   
    "workers": 1,   
    "prefer_ipv6": false
}

```

保存之后去开启一下服务
ssserver -c /etc/shadowsocks/config.json -d start     // 后台启动   不加-d可以看见每次请求

然后就可以肆无忌惮安全上网了. 安卓可以利用ssr去连接.