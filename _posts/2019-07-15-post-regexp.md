---
layout: post
title: vim，grep，sed工具下使用正则表达式的异同
categories:
  - regular expression
tags:
  - regular expression
---

正则表达式在日常工作中可谓是使用得非常广泛了，正常使用正则表达式相信大部分人都没有问题，困扰的是，各种工具使用正则表达式的细节上略有不同，从而导致无法高效地查询、修改我们的文本。下面将介绍三种常用工具中的正则表达式使用上的一些令人困惑的地方

## grep
`grep`命令相信是linux下人们最常使用的命令之一，个人认为使用`grep`中的正则是最简单，也是最接近正则表达式原始定义的，注意到，`grep`使用时可以添加`-E`或者`-P`两个参数，分别表示使用扩展正则表达式和Perl正则表达式，选择自己熟悉的正则规则，相信高效使用`grep`命令没有任何问题

## vim
关于vim编辑器，其主要困惑点在于其中有个`magic`的设定，设定方法为
```
:set magic ; 启用magic，除了 `$`, `.`, `*`, `^` 之外的元字符都要加反斜杠，该设置是默认设置
:set nomagic ; 取消magic，除了 `$`, `^` 之外的元字符都要加反斜杠
:h magic ; 查看帮助
```

另外还可以通过临时开关切换`\m`,`\M`,`\v`,`\V`,加了参数后面的正则表达式会有特殊的magic设置
```
\m: 后面的正则表达式会按照magic处理
例如：/\m.* ; 查找任意字符串

\M: 后面的正则表达式按照nomagic处理
例如：/\M.* ; 查找字符串`.*`(即点后面跟一个星号)

\v: `very magic`，任何元字符都不需要加反斜杠
\V: `very nomagic`，任何元字符都需要加反斜杠
```

注：vim中的变量替换使用的是 \1, \2 分别代表第几个模式

## sed
sed命令可以说与vim非常的类似(基本等同于vim下的`magic`模式)，也需要大量对元字符添加`\`进行转义，并且相比之下，`sed`的正则元字符更少，例如常用的元字符`\w`,`\w`等均无法使用，需要使用`[0-9a-zA-Z]`或者关键词来代替，gnu下的sed的可用字符类关键词如下所示

|  字符类   | 描述  |
|  ----  | ----  |
| [[:alnum:]]  | 字母(a - z A-Z 0 - 9) |
| [[:alpha:]]  | 字母(a - z A-Z) |
| [[:blank:]] | 空白字符(空格或制表键) |
| [[:cntrl:]] | 控制字符 |
| [[:digit:]]	| 数字[0 - 9] |
| [[:graph:]]	| 任何可见字符(不包括空格) |
| [[:lower:]]	| 小写字母的[a -ž] |
| [[:print:]]	| 可打印字符(无控字符) |
| [[:punct:]]	| 标点字符 |
| [[:space:]]	| 空白 |
| [[:upper:]]	| 大写字母的[A -Z] |
| [[:xdigit:]]  | 十六进制数字[0 - 9 a - f A-F] |

注：sed中的变量替换使用的是 \1, \2 分别代表第几个模式

> 参考资料：https://www.jianshu.com/p/3abd6fbc3322