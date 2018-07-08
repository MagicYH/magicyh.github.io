---
layout: post
title: yum命令查看包可用版本列表及安装指定版本
categories:
  - Linux
tags:
  - Linux
---

yum安装软件时默认安装能够提供的最新版本，但是有时候我们并不希望安装最新的版本，这时候我们需要查看有哪些版本可用，然后再安装指定的版本

执行命令，查看可用软件版本，以PHP为例
`yum list addops-php --showduplicates`

结果
```
Installed Packages  
addops-php.x86_64       5.5.25-3.el6        @ADDOPS-base
Available Packages
addops-php.x86_64       5.2.17-1.el6        ADDOPS-base 
addops-php.x86_64       5.2.17-2.el6        ADDOPS-base 
addops-php.x86_64       5.3.27-2.el6        ADDOPS-base 
```


接下来指定安装某个版本，例如php5.3
`yum install addops-php-5.3.27-2.el6`