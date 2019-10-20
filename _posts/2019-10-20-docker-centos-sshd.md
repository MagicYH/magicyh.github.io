---
layout: post
title: Docker制作centos的sshd镜像
categories:
  - docker
tags:
  - docker
---

不啰嗦，直接上Dockerfile讲解
```
// 使用官方centos:7镜像
FROM centos:7

// 从环境变量中读取、设置root密码
ENV ROOT_PASSWORD 123456

// 安装sshd服务
RUN yum install -y openssh-server; \
yum clean all; \

// 生成启动sshd服务需要的秘钥，并且修改root密码
sshd-keygen; \
echo "root:${ROOT_PASSWORD}" | chpasswd

// 暴露sshd服务端口
EXPOSE 22

// 启动服务
CMD ["/usr/sbin/sshd", "-D"]
```

实际上在dockerhub上centos的官方镜像是有说明如何创建一个systemd服务的，但是遗憾的是官方提供的方法不知道什么原因是有问题的，会报类似`Failed to mount tmpfs at /run: Operation not permitted [!!!!!!]`这样的错误，并且无法正常提供服务，估计原因是官方提供的方法使用到了一些额外的权限，但是需要附加的命令以开启权限，因此按照官方的方式正常启动是不行的

因此，这里直接启动sshd服务，安装`openssh-server`后查看`/etc/ssh/sshd_config`的配置，其中可以看到，启动配置需要配置三个key，`rsa, ecdsa, ed25519`，这几个key可以通过`sshd-keygen`命令生成（直接启动sshd的话也会提示这几个key不存在）
```
# 省略……

HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_dsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

# 省略……
```

参考文档：
> [https://www.cnblogs.com/newguy/p/9478455.html](https://www.cnblogs.com/newguy/p/9478455.html)