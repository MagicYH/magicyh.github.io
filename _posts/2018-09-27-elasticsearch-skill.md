---
layout: post
title: ElasticSearch的一些使用小技巧
categories:
  - ElasticSearch
tags:
  - ElasticSearch
---

以前虽然曾经使用过`ElasticSearch`但实际上还不曾深入的研究过，仅仅是作为mysql的一个代替品，用于某些字段的快速检索，如今刚好有一个项目，要在ES上做相对复杂的数据更新，现将在网上查到的内容记录如下

## 1.ES数组的使用
ES本身是支持数组的，或者说ES中的每一个字段都可以保存成0个、1个或者以上元素的数组。只是有一点需要特别注意，**数组类型中各个元素的类型必须相同**。我们可以通过以下命令插入一条新数据

```
curl -X PUT 'localhost:9200/test/persion/5' -d '{
    "name": {
        "first": "magic",
        "last": "orc"
    },
    "age": 26,
    "tags": [
        "ccc",
        "ddd"
    ]
}'
```

其中，`tags`字段就是数组表示

对于数组内容的检索，与正常的表示方法并没有太大区别，使用关键字`terms`进行检索，这里的`terms`相当于mysql中的`in`
```
curl 'localhost:9200/test/persion/_search' -d '{
    "query": {
        "terms": {
            "tags": [
                "ccc",
                "bbb"
            ]
        }
    }
}'
```

## 2.ES对象数组
这种类型我们也经常会用到，比如之前的例子里面，`persion`的`name`属性有`first`和`last`两个子属性构成，在ES中会被处理成以点`.`分隔的字段，即
```
{
    "name.first":"magic",
    "name.last":"orc"
}
```

因此如果需要对子字段执行搜索，则直接用这种形式即可
```
curl 'localhost:9200/test/persion/_search' -d '{
    "query": {
        "term": {
            "name.first": "magic"
        }
    }
}'
```

## 3.ES对象
第二点中提到的对象实际上不是真正的对象，而是扁平化的处理。实际上ES支持在文档中使用完整的子对象，其嵌套的子对象也会被索引为单个的隐藏文档，这种隐藏的对象会能够保存隐藏对象各个属性之间的联系。使用这种功能的时候需要在创建数据类型的时候将数据类型设置为`nested`，另外，查询查询的方法与对象数组是一样的

首先需要设置属性类型为`nested`
```
curl -X PUT 'localhost:9200/test_nested' -d '{
    "mappings": {
        "persion": {
            "properties": {
                "user": {
                    "type": "nested"
                }
            }
        }
    }
}'
```
然后插入元素
```
curl -X PUT 'localhost:9200/test_nested/persion/1' -d '{
    "user": {
        "first": "John",
        "last": "Smith"
    }
}'
curl -X PUT 'localhost:9200/test_nested/persion/2' -d '{
    "user": {
        "first": "Alice",
        "last": "White"
    }
}'
curl -X PUT 'localhost:9200/test_nested/persion/3' -d '{
    "user": [
        {
            "first": "Alice",
            "last": "White"
        },
        {
            "first": "John",
            "last": "Smith"
        }
    ]
}'
```

然后查询
```
curl -X GET "localhost:9200/test_nested/_search" -H 'Content-Type: application/json' -d '{
    "query": {
        "nested": {
            "path": "user",
            "query": {
                "bool": {
                    "must": [
                        {
                            "match": {
                                "user.first": "Alice"
                            }
                        },
                        {
                            "match": {
                                "user.last": "Smith"
                            }
                        }
                    ]
                }
            }
        }
    }
}'

curl -X GET "localhost:9200/test_nested/_search" -H 'Content-Type: application/json' -d '{
    "query": {
        "nested": {
            "path": "user",
            "query": {
                "bool": {
                    "must": [
                        {
                            "match": {
                                "user.first": "Alice"
                            }
                        },
                        {
                            "match": {
                                "user.last": "White"
                            }
                        }
                    ]
                }
            }
        }
    }
}'
```

这里可以对比一下`nested`和对象数组的区别，`nested`保留了对象各属性之间的关系，而对象数组则更像是把各个元素的属性组织成了数组
```
// 插入元素
curl -X PUT 'localhost:9200/test/persion/5' -d '{
    "user": {
        "first": "John",
        "last": "Smith"
    }
}'
curl -X PUT 'localhost:9200/test/persion/6' -d '{
    "user": {
        "first": "Alice",
        "last": "White"
    }
}'
curl -X PUT 'localhost:9200/test/persion/7' -d '{
    "user": [
        {
            "first": "Alice",
            "last": "White"
        },
        {
            "first": "John",
            "last": "Smith"
        }
    ]
}'

// 检索
curl -X GET "localhost:9200/test/_search" -H 'Content-Type: application/json' -d '{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "name.first": "Alice"
                    }
                },
                {
                    "match": {
                        "name.last": "White"
                    }
                }
            ]
        }
    }
}'

curl -X GET "localhost:9200/test/_search" -H 'Content-Type: application/json' -d '{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "name.first": "Alice"
                    }
                },
                {
                    "match": {
                        "name.last": "Smith"
                    }
                }
            ]
        }
    }
}'
```

## 4.ES更新
ES的基础更新操作看似功能比较少，每次更新操作必须要覆盖操作。而实际上ES内置了通过`script`的方式进行更新，也就是说更新的时可以执行一个脚本来进行更新，内置的脚本语言是`Groovy`（官网的最新文档没找到那个章节写了这个，是在旧的中文翻译文档里提到的），以下是一些基础功能的写法，更多高级用法可以参考`Groovy`脚本语言

```
// 更新某个数值
curl -X POST "localhost:9200/test/persion/1/_update" -H 'Content-Type: application/json' -d '{
    "script": {
        "source": "ctx._source.age += params.age",
        "lang": "painless",
        "params": {
            "age": 2
        }
    }
}'

// 增加标签
curl -X POST "localhost:9200/test/persion/1/_update" -H 'Content-Type: application/json' -d '{
    "script": {
        "source": "ctx._source.tags.add(params.tag)",
        "lang": "painless",
        "params": {
            "tag": "blue"
        }
    }
}'

// 按条件删除
curl -X POST "localhost:9200/test/persion/1/_update" -H 'Content-Type: application/json' -d '{
    "script": {
        "source": "if (ctx._source.tags.contains(params.tag)) { ctx.op = 'delete' } else { ctx.op = 'none' }",
        "lang": "painless",
        "params": {
            "tag": "red"
        }
    }
}'

// 不存在则添加，存在则移除已存在的项
curl -X POST "localhost:9200/test/persion/1/_update" -H 'Content-Type: application/json' -d '{
    "script": {
        "source": "if ( !ctx._source.tags.contains(params.tag) ) {  ctx._source.tags.add(params.tag) } else {  ctx._source.tags.remove(ctx._source.tags.indexOf(params.tag)) }",
        "lang": "painless",
        "params": {
            "tag": "blue"
        }
    }
}'
```

### script更新时遇到的一个错误
```
// 当执行以下更新语句时遇到一个错误，以下更新的脚本的目的是，当`first = Alice`的用户存在时更新他的last信息，如果不存在则追加一个用户信息
curl -X POST "127.0.0.1:9200/test_nested/persion/3/_update" -d '
{
    "script": {
        "source": "int idx = -1;for(int i = 0; i < ctx._source.user.size(); i++) { if (ctx._source.user[i].first == params.first) { idx = i; break; }} if ( idx > -1) { ctx._source.user[idx] = params } else { ctx._source.user.add(params) } ",
        "lang": "painless",
        "params": {
            "first": "Alice",
            "last": "Smith"
        }
    }
}'

// 执行后会出现如下错误
{"error":{"root_cause":[{"type":"remote_transport_exception","reason":"[uYVDMhK][127.0.0.1:9300][indices:data/write/update[s]]"}],"type":"illegal_argument_exception","reason":"Iterable object is self-referencing itself"},"status":400}
```

最初真的是看不明白这个错误到底是什么意思，经过一番google，终于找到一个有点意思的说法
> 参考：[https://github.com/elastic/elasticsearch/issues/19475](https://github.com/elastic/elasticsearch/issues/19475)

大意是说，`params`这个参数下面，除了我们传参传进去的`first、last`两个参数之外，`ctx`这个参数其实也包括在`params`之中，也就是说`params->ctx, params->first, params->last`都是`params`中的参数，因此，直接把`params`赋值给`ctx._source.user`的这个操作可能在某处引起了无限循环，从而产生这个错误，正确的方法应当是在params中再包一层参数即可，例如：
```
curl -X POST "127.0.0.1:9200/test_nested/persion/3/_update" -d '
{
    "script": {
        "source": "int idx = -1;for(int i = 0; i < ctx._source.user.size(); i++) { if (ctx._source.user[i].first == params.user.first) { idx = i; break; }} if ( idx > -1) { ctx._source.user[idx] = params.user } else { ctx._source.user.add(params.user) } ",
        "lang": "painless",
        "params": {
            "user": {
                "first": "Alice",
                "last": "Smith"
            }
        }
    }
}'
```

## 5.ES更新索引类型
使用如下语句进行mapping的更新，下例增加一个`id`字段
```
curl -X POST "127.0.0.1:9200/test_nested/persion/_mapping" -d '
{
    "persion": {
        "properties": {
            "id": {
                "type": "integer"
            }
        }
    }
}'
```

> 参考资料：<br>
> [http://www.cnblogs.com/ljhdo/p/4904430.html](http://www.cnblogs.com/ljhdo/p/4904430.html) <br>
> [https://www.elastic.co/guide/en/elasticsearch/reference/current/nested.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/nested.html) <br>
> [https://es.xiaoleilu.com/052_Mapping_Analysis/50_Complex_datatypes.html](https://es.xiaoleilu.com/052_Mapping_Analysis/50_Complex_datatypes.html) <br>
> [https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-update.html) <br>
> [https://es.xiaoleilu.com/030_Data/45_Partial_update.html](https://es.xiaoleilu.com/030_Data/45_Partial_update.html)