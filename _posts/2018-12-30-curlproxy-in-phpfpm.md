---
layout: post
title: 在php-fpm中设置curl全局代理
categories:
  - php-fpm
tags:
  - php-fpm
  - curl
  - proxy
---

首先描述一下应用背景，首先正常环境下肯定是不需要这么麻烦的设定，正常在代码中使用curl库就行了。但是在公司特殊的网络环境下，有些服务器无法直接访问外网，必须通过代理。而代码中有些业务又必须访问外网，为了使环境尽量保持一致而无需在代码层面进行兼容，在无法直接访问外网的机器上部署时，需要配置代理，使得从php-fpm中执行的代码可以按照规则通过代理转发请求

### 配置方法
配置方法很简单，就是使用环境变量。与bash中设置代理的方法和变量，主要使用`http_proxy`, `https_proxy`, `no_proxy`三个变量，变量的含义也与bash中一致

假设某个php-fpm的池子的配置文件在`$PHP_ROOT/etc/php-fpm.d/www.conf`文件中，可以为这个池子加上全局代理，在配置文件末尾加上环境变量
```
env[http_proxy] = http://proxy_host:[proxy_port]
env[https_proxy] = https://proxy_host:[proxy_port]
env[no_proxy] = *.oa.com,localhost,127.0.0.1,172.17.0.1
```

加上以上配置以后重启php-fpm，则在代码中使用curl访问站点时，会自动使用代理而不需要再代码中显式地调用代理。

其中`no_proxy`这个变量需要注意。这个变量采用的前缀匹配规则，也就是说匹配上后缀的域名将不会使用全局代理(是否是这种写法还要确认，我没有确认过这样写是否是能够通用匹配的)。对于`no_proxy`中的ip地址需要十分注意，这个变量是不能接受ip范围配置的，只能将所有想要禁用代理的地址全部写入到no_proxy变量中(大概这个变量主要还是要让我们使用域名方式的)。网上有些写法例如：`10.0.*.*, 10.0., 10.0.0.0/16`这些写法**全部是不可用的！！**，至少在我的系统上确认是这样。

> 参考资料：https://unix.stackexchange.com/questions/23452/set-a-network-range-in-the-no-proxy-environment-variable