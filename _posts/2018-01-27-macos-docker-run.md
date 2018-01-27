---
layout: post
title: MacOS Docker启动不能正确绑定80端口的问题
categories:
  - Docker
tags:
  - Docker
---

第一次使用Docker启动镜像，并且绑定80端口，发现一段诡异的报错

`Bind for 0.0.0.0:80: unexpected error (Failure EADDRINUSE)`

使用命令：`netstat -an | grep 80` 发现80端口已经被占用了，导致Docker无法正常映射到80端口，根据资料，是由于apache导致的（Mac上默认会启动Apache服务？）

使用命令：`sudo /usr/sbin/apachectl stop` 结束掉服务，然后再重新启动Docker容器即可

> 参考资料：https://github.com/Islandora-Collaboration-Group/ISLE/issues/31