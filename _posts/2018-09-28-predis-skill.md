---
layout: post
title: predis的一个使用小技巧
categories:
  - predis
tags:
  - predis
---

laravel框架默认使用`predis`作为`redis`的默认驱动，但`predis`的文档不太全，简单命令还好，有些带参数的命令其预设的函数不支持选项，查了半天文档，终于找到一个大杀器命令，`executeRaw`，该命令接受两个参数，第一个参数是数组，按照标准的redis命令输入即可，第二个参数是引用参数，如果发生错误会返回错误响应

该函数的原型在文件`predis/src/Client.php`内，使用示例如下
```
$client->executeRaw(['SET', 'myset', 'hello', 'NX', 'EX', '3600'], $error)

// 其次，上面的命令可以用以下命令来代替
$client->set($key, 0, 'NX', 'EX', $ex)
```

虽然这样，如果有可能的话还是建议使用phpredis扩展吧

> 参考资料：<br>
> [http://mjiong.com/docs/20062.html#i-6](http://mjiong.com/docs/20062.html#i-6)