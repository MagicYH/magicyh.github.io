---
layout: post
title: 使用curl查看请求中各部分耗时
categories:
  - curl
tags:
  - curl
---

curl命令想必大家或多或少都用过，大部分情况下我们只使用其最简单的功能，通过curl命令访问某个地址。实际上curl是一个功能非常强大的工具，我们可以利用它来分析某个请求在各个步骤上的耗时。

分析各个步骤的耗时主要使用到了`curl`的`-w`参数，使用`man`命令查看，该参数的主要描述如下
```
-w, --write-out <format>
        Defines what to display on stdout after a completed and successful operation. The format is a string that may contain plain text mixed with any number of variables. The
        string can be specified as "string", to get read from a particular file you specify it "@filename" and to tell curl to read the format from stdin you write "@-".

        The  variables  present  in  the output format will be substituted by the value or text that curl thinks fit, as described below. All variables are specified as %{vari‐
        able_name} and to output a normal % you just write them as %%. You can output a newline by using \n, a carriage return with \r and a tab space with \t.

        NOTE: The %-symbol is a special symbol in the win32-environment, where all occurrences of % must be doubled when using this option.
```

其中提供的可以输出的参数有很多，具体可以自己在`manpage`中查找，首先往`curl-format.txt`文件中写入我们关心的耗时
```
    time_namelookup:  %{time_namelookup}\n
       time_connect:  %{time_connect}\n
    time_appconnect:  %{time_appconnect}\n
      time_redirect:  %{time_redirect}\n
   time_pretransfer:  %{time_pretransfer}\n
 time_starttransfer:  %{time_starttransfer}\n
         time_total:  %{time_total}\n
```

简单解释一下这些变量的含义
- time_namelookup: 域名解析消耗的时间
- time_connect: TCP建立连接消耗的时间，也就是三次握手的耗时
- time_appconnect: SSL/SSH等上层协议的耗时
- time_redirect: 重定向的耗时，包括域名解析，连接建立等。直到最终连接之前的耗时
- time_pretransfer: 从请求开始到文件开始传输前的总耗时
- time_starttransfer: 从请求开始到第一个字节开始传输前的总耗时，包括time_protransfer和服务器需要做一些计算的时间
- time_total: 传输总耗时

更多详细参数可以查看manpage，具体使用示例如下
```
$ curl -w "@curl-format.txt" -o /dev/null -s -L "https://imgnode.gtimg.cn/hq_img?type=minute&size=1&proj=financehome&code=hkHSI" >> time_consume
$ cat time_consume
    time_namelookup:  3.002
       time_connect:  3.008
    time_appconnect:  3.103
      time_redirect:  0.000
   time_pretransfer:  3.103
 time_starttransfer:  3.166
         time_total:  3.171
```

这个命令在分析请求故障的时候很有用，某一次在发现一个问题，在某台服务器上，使用php-curl下载文件时总是失败。但是登录服务器上，直接使用wget命令下载时是成功的。添加日志以后发现curl的报错为`name lookup timed out`。在网上检索这个错误以后，发现是域名解析的问题。因为curl会优先查找IPV6的域名，如果服务器支持IPV6(这个部分参考了网上的资料，不明确是那个服务支持IPV6，是DNS服务器还是指调用curl的服务器，我使用的服务器是没有IPV6地址的，所以考虑这里指DNS服务器的概率比较大)，curl会先解析IPV6，再解析IPV4从而容易导致超时。命令行下使用curl命令分析了个步骤的耗时后也确认了这个判断，域名解析耗时超过3秒。

最后解决问题的方法有三种：
1. 代码中的curl强制访问IPV4，使用如下代码`curl_setopt($ch, CURLOPT_IPRESOLVE, CURL_IPRESOLVE_V4);`
2. 修改域名解析服务器，修改`/etc/resolv.conf`文件的配置，在第一行写入例如：`nameserver 8.8.8.8`(越靠前的DNS越优先被使用)
3. 如果有IPV6网络可以尝试禁用IPV6网络，但这种方法实在是不推荐


> 参考资料 <br>
> https://cizixs.com/2017/04/11/use-curl-to-analyze-request/ <br>
> https://blog.josephscott.org/2011/10/14/timing-details-with-curl/ <br>
> https://www.04007.cn/article/408.html