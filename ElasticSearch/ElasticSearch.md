# ElasticSearch

> The Elastic Stack, 包括 Elasticsearch、Kibana、Beats 和 Logstash（也称为 ELK Stack）。 能够安全可靠地获取任何来源、任何格式的数据，然后实时地对数据进行搜索、分析和可视 化。Elaticsearch，简称为 ES，**ES 是一个开源的高扩展的分布式全文搜索引擎**，是整个 Elastic  Stack 技术栈的核心。它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上 百台服务器，处理 PB 级别的数据。



### 数据格式

![](C:\Users\13218\Desktop\study\ElasticSearch\image\屏幕截图 2024-01-29 230847.png)

ES 里的 Index 可以看做一个库，而 Types 相当于表，Documents 则相当于表的行。 这里 Types 的概念已经被逐渐弱化，Elasticsearch 6.X 中，一个 index 下已经只能包含一个 type，Elasticsearch 7.X 中, Type 的概念已经被删除了。

### 倒排索引

> 与倒排索引相对的是正排索引，它就是我们通常见到的索引，一个字段作为索引，然后通过这个字段索引可以快速查询到数据。
>
> 但是如果我们要进行与数据本身相关的查询，一种办法是模糊查询，但是这种查询方式效率很低，索引可能会失效。



### 启动 ES

在windows系统中进入到bin目录，打开可执行文件`elsticsearch.bat`

## 二、ES基础操作

### 2.1 索引操作

> 我们之前讲过，ES中的索引相当于mysql数据库中的Database，所以我们要首先创建索引。

#### 2.1.1 创建索引

这个操作是幂等性的

```json
发送 PUT 请求：http://127.0.0.1:9200/shopping

--------响应-------------
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "shopping"
}
```

#### 2.1.2 操作索引

```json
-------获取指定索引信息-------
发送 GET 请求：http://127.0.0.1:9200/shopping
{
 	"shopping"【索引名】: { 
 		"aliases"【别名】: {},
 		"mappings"【映射】: {},
 		"settings"【设置】: {
 			"index"【设置 - 索引】: {
 				"creation_date"【设置 - 索引 - 创建时间】: "1614265373911",
				 "number_of_shards"【设置 - 索引 - 主分片数量】: "1",
 				"number_of_replicas"【设置 - 索引 - 副分片数量】: "1",
 				"uuid"【设置 - 索引 - 唯一标识】: "eI5wemRERTumxGCc1bAk2A",
                 "version"【设置 - 索引 - 版本】: {
 					"created": "7080099"
 				},
 				"provided_name"【设置 - 索引 - 名称】: "shopping"
 			}
 		}
	 }
}
-------删除指定索引-------
发送 DELETE 请求：http://127.0.0.1:9200/shopping

-------获取所有索引信息-------
GET http://127.0.0.1:9200/_cat/indices?v
```



### 2.2 文档操作

> 文档相当于关系型数据库中的行。添加数据

#### 2.2.1 创建文档

索引已经创建好了，接下来我们来创建文档，并添加数据。这里的文档可以类比为关系型数 据库中的表数据，添加的数据格式为 JSON 格式 在 Postman 中，向 ES 服务器发 POST 请求

这个操作不是幂等性的，可以多次操作

```json
POST http://127.0.0.1:9200/shopping/_doc
------自定义id-------
POST http://127.0.0.1:9200/shopping/_doc/1001
请求体内容为：
{
 "title":"小米手机",
 "category":"小米",
 "images":"http://www.gulixueyuan.com/xm.jpg",
 "price":3999.00
}
```

#### 2.2.2 操作文档

```json
-------根据指定id查询----------
GET http://127.0.0.1:9200/shopping/_doc/1001
-------查询指定索引下的所有文档---------
GET http://127.0.0.1:9200/shopping/_search

-------完全覆盖指定文档数据（幂等操作）--------------
PUT http://127.0.0.1:9200/shopping/_doc/1001
{
 "title":"小米手机",
 "category":"小米",
 "images":"http://www.gulixueyuan.com/xm.jpg",
 "price":4999.00
}
---------局部更新指定文档数据（非幂等操作），再次修改会返回noop-----
POST http://127.0.0.1:9200/shopping/_update/1001
{
    "doc":{
        "title":"华为手机"
    }
}
----------删除指定id文档------
DELETE http://127.0.0.1:9200/shopping/_doc/1001
```

#### 2.2.3 全查询

```json
GET http://127.0.0.1:9200/shopping/_search
{
    "query":{
        "match_all":{
            
        }
    },
    "from":0,
    "size":1,
    "_source":["title"],
    "sort":{
        "price":{
            "order":"desc"
        }
    }
}
----------------
match: 匹配规则
from: 起始页码
size: 每页多少数据
_source:指定显示字段
sort: 排序
```



#### 2.2.4 多条件查询

```json
{
    "query":{
        "bool":{
            "must":[
                {
                    "match":{
                        "category":"小米"
                    }
                },
                {
                	"match":{
                		"price":3999.00
                }
                }
            ]
        }
    }
}
```

#### 2.2.5 全文检索

> es会将搜索的关键词进行分词，然后分别查询，如果要是完全匹配要使用`match_phrase`字段



#### 2.2.6 聚合操作

> 查询结果有原始数据+ 统计结果，加上size字段可以只看聚合结果

```json
{
    "aggs": {//聚合操作
        "price_group":{//分组名称
            "terms":{// 分组
                "field":"price" //分组字段
            }
        }
    },
    "size":0
}
```



#### 2.2.7 映射关系

 ```json
 PUT http://127.0.0.1:9200/user/_mapping
 {
     "properties":{
         "name":{
             "type":"text",
             "index":true
         },
         "sex":{
             "type":"keyworkd",
             "index":true
         },
         "tel":{
             "type":"keyword",
             "index":false
         }
     }
 }
 
 --------------------------
 name 字段可以被分词查询
 sex 字段类型为keyword关键字，不能被分词查询
 tel 字段为不能被索引查询到
 ```



### 2.3 JavaAPI





