---
layout: post
title: vscode golang 插件安装
categories:
  - vscode
tags:
  - vscode
---

vscode是目前比较好用的一款编辑器，为了能够充分发挥其功能，需要安装不少的插件。但是因为墙的原因，golang的vscode插件不能够直接安装，因此需要做一些准备

1. 进入GOPATH目录
2. 创建文件夹`mkdir -p src/golang.org/x`
3. 进入刚才创建的目录`cd src/golang.org/x`
4. 手动下载依赖的代码
    - `git clone  https://github.com/golang/tools.git tools`
    - `git clone  https://github.com/golang/lint.git lint`
5. 做好以上准备以后，即可让vscode自行安装所需的插件
6. 如果还有插件安装失败的话可以自己执行命令进行重试`go install ${dependency}`