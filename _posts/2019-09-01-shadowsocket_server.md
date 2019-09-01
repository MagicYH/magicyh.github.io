---
layout: post
title: shadowsocket服务端搭建教程
categories:
  - shadowsocket
tags:
  - shadowsocket
---

一般折腾这个就是为了能够翻墙，不想折腾的话直接买VPN就好了，本文仅针对目前持有能够访问海外站点的服务器的人群，具体服务器的获取方式有很多（收费的），大部分云服务提供商都有，国外的例如亚马逊、谷歌，国内的阿里云，腾讯云等

## 搭建shadowsocket服务器
1. 安装`python-pip`
一般云服务器目前使用的系统多数为`centos`，在此以`centos`为例来说明怎么安装`pip`

```
# 检索pip包名，因为各个厂商依赖的源可能会不同（特别国内厂商可能是自建源），因此需要看看`pip`的包名是什么
yum search pip 

# 假设检索到的包名为: python2-pip，使用如下命令进行安装
yum install -y python2-pip
```

2. 配置服务
在服务器上创建配置文件`/etc/shadowsocks.json`(注意这个文件可以修改位置，并不是严格要求的，启动服务端时可以选择配置文件路径)，配置内容如下

```
{
  "server": "0.0.0.0",
  "server_port": 8989,          // 服务器对外暴露的端口
  "local_address": "127.0.0.1",
  "local_port": 1080,
  "password": "yourpassword",   // 登录的密码
  "timeout": 300,
  "method": "aes-256-cfb",      // 加密方式，客户端配置时需要与这个保持一致
  "fast_open": false
}
```

也可以配置多个端口号

```
{
  "server": "0.0.0.0",
  "local_address": "127.0.0.1",
  "local_port": 1080,
  "port_password": {
    "8989": "password0",        // 每个端口号对应一个密码，如此这个服务可以提供给多个用户使用
    "9001": "password1",
    "9002": "password2",
    "9003": "password3",
    "9004": "password4"
  },
  "timeout": 300,
  "method": "aes-256-cfb",
  "fast_open": false
}
```
需要注意的一点是，必须要确保对外暴露的端口可以通过外网访问到(如果不通的话可能要查看一下安全策略，例如防火墙等等，把可以对外暴露的端口打开)

3. 启动服务
```
ssserver -c /etc/shadowsocks.json -d start
```

4. 添加到开机启动项(可选)
```
echo "ssserver -c /etc/shadowsocks.json -d start" >> /etc/rc.local
```

## 客户端配置
需要配置的项包括
1. 服务器地址外网IP和配置中暴露的端口号，以之前的配置为例就是`host:8989`
2. 选择加密方式，以之前配置为例就是`aes-256-cfb`
3. 设置密码，以之前的配置为例就是`yourpassword`


## 参考资料
> http://kids.codepku.com/topic/view/866
> https://www.banpie.info/shadowsocks-pac-gfw/