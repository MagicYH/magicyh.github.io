---
layout: post
title: Ubuntu14.04 jekyll 环境部署
---

Jekyll可以用于在github上搭建博客，因为Ubuntu14.04上自带的ruby版本比较低，这里记录一下如何在Ubuntu上搭建一个jekyll可用的环境

### 删除旧版本的ruby
首先使用命令，查看系统上都有哪些ruby的相关应用，只用使用apt命令全部删除，避免版本混乱造成不必要的麻烦
``` bash
# 查看ruby相关包
dpkg -l | grep ruby

# 删除ruby相关包
sudo apt-get autoremove ruby

# 如果发现还有残留的配置文件剩余，则使用如下命令清除
dpkg --purge ruby
```

### 安装rvm
`rvm`是ruby的版本管理工具，安装以后便于各个版本的ruby的安装与切换
``` bash
# 需要使用curl命令，如果没有请使用apt命令安装
sudo apt-get install curl

# 安装rvm，官方推荐的方式为curl -L get.rvm.io | bash -s stable，如果遇到错误，则可以根据提示解决
curl -L get.rvm.io | bash -s stable

# rvm最终会安装在 source $home/.rvm/scripts/rvm 目录下（其中$home表示当前用户目录），同时需要修改bashrc文件
echo 'source $home/.rvm/scripts/rvm' >> ~/.bashrc

# 执行命令，如果能够正确执行则表示rvm安装成功了
rvm -v
```

### 安装ruby
``` bash
# 首先执行命令，查看可用的ruby版本
rvm list know

# 以2.4为例，安装ruby
rvm install 2.4

# 执行命令以检查ruby是否正确安装
gem -v
```

### 安装jekyll环境
``` bash
# 使用gem安装jekyll，可能需要sudo
gem install jekyll

# 安装bundle
gem install bundle

# 进入模板项目根目录，如果存在Gemfile.lock文件，先将其删除
rm Gemfile.lock
bundle install

# 最后运行jekyll
jekyll serve -H 127.0.0.1
```