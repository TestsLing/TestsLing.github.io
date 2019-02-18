---
layout:     post
title:      "Elasticsearch搜索服务"
subtitle:   "强大的Elasticsearch"
date:       2019-02-18 14:14:00
author:     "憧憬"
header-img: "img/about-bg-walle.jpg"
header-mask: 0.3
catalog:    true
tags:
  - Elasticsearch
---
## elasticsearch 安装

```
在bin 下面有个脚本   
./bin/elasticsearch  -d 是后台运行
```



## elasticsearch-head 集群管理

+ 修改config 下面的yml文件 解决跨域

```
http.cors.enabled: true
http.cors.allow-origin: "*"
```



+ 然后使用head插件就能检测到这个服务的运行

```
head 插件安装 GitHub搜索 elasticsearch   然后下载下来 npm insstall 
安装完后 npm run start

head里面有一个集群健康值:
绿色是健康
黄色 所有的主分片可用，但是部分副本分片不可用
红色 此时执行查询部分数据仍然可以查到，可能会造成数据丢失 遇到这种情况，还是赶快解决比较好
```



1. 设置master节点
2. 设置slave节点

+ 这边演示设置1个master节点和2个slave节点

> master节点配置文件修改

```

#允许该节点存储数据(默认开启)
#node.data: true

cluster.name: woailuo     设置集群名称
node.name: master		设置节点名称
node.master: true		是否有资格设置为主节点  如果设置多个true  在这个主节点挂掉之后会自动切换 为true的主节点
network.host: 127.0.0.1  绑定地址

# 设置对外服务的http端口，默认为9200
# http.port: 9200
端口默认是9200 master未做端口更改

# 设置节点间交互的tcp端口,默认是9300 
# transport.tcp.port: 9300

# 如果没有这种设置,遭受网络故障的集群就有可能将集群分成两个独立的集群 - 分裂的大脑 - 这将导致数据丢失
# discovery.zen.minimum_master_nodes: 3

修改完重启一下ES就可以了

在head插件里面也可以看到名称发生改变了
```



> slave节点配置文件修改

```
修改slave配置文件的cluster.name  必须跟主节点一致

cluster.name: woailuo
node.name: slave1   		设置子节点名称
network.host: 127.0.0.1		设置绑定地址
http.port: 8200			    设置这个ES端口    不能设置默认端口了  否则会出现端口冲突
discovery.zen.ping.unicast.hosts: ["127.0.0.1"]    找到master 如果不加这个配置 是找不到集群的

cluster.name: woailuo
node.name: slave2
node.master: true
network.host: 127.0.0.1
http.port: 7200
discovery.zen.ping.unicast.hosts: ["127.0.0.1"]

```

> 设置成功之后  主节点会有一个五角星图标  其余副节点会有圆圈



## ES基础概念

1. 索引

   + 含有相同属性的文档集合

   ```
   ES在创建索引时  默认是创建5个分片 一个备份  这个数量是可以修改的 分片是只能创建时修改  备份可以动态修改
   在索引中  还存在几个概念
   1. 分片
   		每个索引都有多个分片吧,每个分片是一个lucene索引
   2. 备份
   		拷贝一份分片就完成了分片的备份
   		主分片如果损坏  备份的分片还可以提供搜索
   ```

   ​

2. 类型

   + 索引可以定义一个或多个类型,文档必须属于一个类型

3. 文档

   + 文档是可以被索引的基本数据单位

>索引可以看成数据库的库   类型可以看成数据表 文档可以看成表中的某条数据
>
>比如说:
>
>我们存储一个数据有几个大类: 动物 书籍
>
>可以把动物和书籍设置为索引
>
>但是书籍或者动物都有小类别
>
>把这些小类别设置为类型   那么具体的书籍或者动物的信息就是文档

+ 添加索引

  > 添加索引后可以查看索引信息

  1. 结构化

     ​

  2. 非结构化

     mappings后面为{} 则为非结构化

     创建结构化索引

     ```
     http://localhost:9200/book/novel/_mappings    给book索引添加类型
     {
       "novel": {      novel下面创建类型
         "properties": {   
           "title": {
             "type": "text"
           }
         }
       }
     }

     创建索引及类型
     http://localhost:9200/pople 创建peplo索引
     {
     	"settings":{   设置索引分片数量
     		"number_of_shards": 1,
     		"number_of_replicas":1 设置索引备份数量
     	},
     	"mappings":{   设置类型
     		"man":{
     			"properties":{
     				"name":{
     					"type":"text"
     				},
     				"country":{
     					"type":"keyword"
     				},
     				"age":{
     					"type":"date",
     					"format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis 时间戳" 
     				}
     			} 
     		}
     		
     	}
     }
     ```

     ​

+ 插入

  1. 指定文档id插入

     ```
     PUT 方式
     http://localhost:9200/pople/man/1       完全基于resultful API 索引路由格式基本就是这种风格
     					/索引/类型/指定文档id

     根据类型字段发送JSON
     例如:
     {
     	"country": "US",
     	"name": "mike",
     	"age": "2019-07-01"
     }
     ```

     ​

  2. 自动产生文档id插入

     ```
     自动产生文档id需要使用post方式插入
     http://localhost:9200/pople/man
     数据格式还是按照类型写入 即可
     ```

     ​

+ 修改

  1. 直接修改文档

     ```
     http://localhost:9200/pople/man/1/_update   POST

     修改的文本JSON
     {
         "doc":{
             "name":"test"
         }
     }

     ```

     ​

  2. 脚本修改文档

     ```
     http://localhost:9200/pople/man/1/_update   POST

     修改的文本JSON
     {
         "script":{
             "lang":"painless",    ES自带语言  还支持其他脚本语言  例如 python
             "inline":"ctx._source.age += 10  (或者写成  age = parmas.age)",   ctx上下文对象 _source当前文档
             "params":{
                 "age":100
             }
         }
     }
     ```

+ 删除

  1. 删除文档

     ```
     http://localhost:9200/pople/man/1              DELETE方式 直接删除即可
     ```

     ​

  2. 删除索引

     ```
     慎重删除索引
     删除后索引及文档全部删除
     http://localhost:9200/pople              DELETE方式 直接删除即可

     ```

## 查询

+ 简单查询

  ```
  http://localhost:9200/索引/类型/id         GET方式即可
  ```

  ​

+ 条件查询

  ```
  http://localhost:9200/book/_search        POST

  查询JSON
  {
  	"query":{
  		"match_all":{}   查询所有
  	},
  	"from":1,            设置数据偏移量
  	"size":1			设置获取数据条数   结合可做分页
  }


  {
  	"query":{
  		"match":{
              "title":"test"      搜索该索引 类型为title  文档带有test字符的数据
  		}
  	},
  	"sort":[				默认是_score进行排序   我们指定排序 _score属性会变成null
  			{
  				"publish_date":{  以publish_date倒序排序
  					"order":"desc"
  				}
  				
  			}
  		]
  }
  对于match 查询  针对不同的类型查询结果也不一样
   keyword是关键字不可切分的，是全匹配的
   
   使用match_phrase  短语匹配  完整匹配
  ```

  ​

+ 聚合查询

  ```
  {
  	"aggs":{
  		"group_by_word_count":{       分组名称  自定义  可以对多个字段进行分组
  			"terms":{
  				"field":"word_count"
  			}
  		},
  		"group_by_publish_date":{
  			"terms":{
  				"field":"publish_date"
  			}
  		}
  	}
  }


  {
  	"aggs":{
  		"grades_word_count":{
  (可以直接设置成max 或avg min等函数)	"stats":{   计算聚合  可以求平均  最大 最小 求和
  				"field":"word_count"
  			}
  		}
  	}
  }
  ```

+ 自条件查询

  > 特定字段查询所指特定值

  1. query context

     > > 在查询过程中,除了判断文档是否满足查询条件外,ES还会计算一个_score来表示匹配程度,旨在判断目标文档和查询条件匹配有多好

     + 全文本查询

       > 针对文本类型数据

       ```
       {
       	"query":{
       		"multi_match":{
       			"query":"张三",
       			"fields":["author","title"]
       		}
       	}
       }
       多字段查询   


       语法查询

       {
       	"query":{
       		"query_string":{
       			"query":"三 OR JAVA",   可以设置正常查询条件 OR  AND  还可以使用()设置优先级
       			"fields":["author","title"]
       		}
       	}
       }
       ```

       ​

     + 字段级别查询

       > 针对结构化数据 如 数字,日期等

       ```
       {
       	"query":{
       		"term":{
       			"author":"张三"
       		}
       	}
       }
       term是代表完全匹配，也就是精确查询

       范围查询range   gte大于   lte小于  可以设置日期 和数字等

       日期查询
       "get":2017-01-01 
       "lte":now  查询从2017-01-01 到现在时间
       {
       	"query":{
       		"range":{
       	
       				"word_count":{  针对word_count字段 
       					"gte":1000, 
       					"lte":5000
       				}
       			

       		}
       	}
       }
       ```

       ​

       ​

  2. filter context

     > 在查询过程中.只判断该文档是否满足条件,只有Yes和No   而query还会使用分析器去分析匹配程度
     >
     > filter相对query查询较快
     >
     >  filter会自动缓存 需要集合bool一起使用

     ```
     {
     	"query":{
     		"bool":{
     			"filter":{
     				"term":{
     					"word_count":1000
     				}
     			}
     		}
     	}
     }
     ```

     ​

+ 复合条件查询

  > 以一定逻辑组合子条件查询

  1. 固定分数查询

  ```
  {
  	"query":{
  		"constant_score":{   分数查询
  			"filter":{  只支持filter 不能用match
  				"match":{
  					"title":"JAVA"
  				}
  			},
  			"boost":2  设置分数为2
  		}
  	}
  }
  ```

  ​

  2. 布尔查询

  ```
  {
  	"query":{
  		"bool":{
  			"should":[     should是OR条件  满足其中一即可   如果要AND条件 使用must关键词
  				{
  					"match":{
  						"author":"张三"
  					}
  				},
  				{
  					"match":{
  						"title":"JAVA"
  					}
  				}
  			]
  		}
  	}
  }

  {
  	"query":{
  		"bool":{
  			"must":[
  				
  				{
  					"match":{
  						"title":"JAVA"
  					}
  				}
  			],
  			"filter":{    设置多条件  大于1000小于2000
  				"range":{
  					"word_count":{
  						"gte":1000,
  						"lte":2000
  					}
  				}
  			}
  		}
  	}
  }

  只查看author不是张三的
  {
  	"query":{
  		"bool":{
  			"must_not":{
  				"term":{
  					"author":"张三"
  				}
  			}
  		}
  	}
  }
  ```

  ​