### 插入一条数据

```
PUT /megacorp/employee/1
{
"first_name" : "John",
"last_name" : "Smith",
"age" : 25,
"about" : "I love to go rock climbing",
"interests": [ "sports", "music" ]
}
```

| megacorp | 索引名字（数据库） |
| -------- | ------------------ |
| employee | 类型名字（表）     |
| 1        | 当前员工id         |

### 批量插入数据可通过json文件进行上传导入（Postman）

```
192.168.17.129:9200/_bulk
```

数据格式

```
{"index" : {"_index":"book","_id":"1","_type":"test"}}
{"name":"三体","price":"66"}
{"index" : {"_index":"book","_id":"2","_type":"test"}}
{"name":"三体2","price":"66"}
{"index" : {"_index":"book","_id":"3","_type":"test"}}
{"name":"三体3","price":"66"}

```

注意：奇数行为索引和type 偶数行为数据 并且利用post请求导入数据文件

#### 查询https协议资产

```
{
    "query":{
        "match":{
            "protocol":"https"
        }
    },
    "_source":[
        "ip","port","protocol","banner","cert"
    ],
    "size": 20
}
```

> 注：如果想精确匹配，在传入参数是字段加入.keyword,使用term或match_phrase,因为存在分词现象
>
> term做精确查询可以用它来处理数字，布尔值，日期以及文本。查询数字时问题不大，但是当查询字符串时会有问题。term查询的含义是termQuery会去倒排索引中寻找确切的term,但是它并不知道分词器的存在。term表示查询字段里含有某个关键词的文档，terms表示查询字段里含有多个关键词的文档。也就是说直接对字段进行term本质上还是模糊查询，只不过不会对搜索的输入字符串进行分词处理罢了。
>
> `term` 是精确搜索，搜索的时候会将用户的搜索内容，比如"好的"作为一整个关键词去搜索，而不会对其进行分词后再搜索。

```
{
    "query": {
        "term": {
            "desc": "好的"
        }
    },
        "_source": [
            "id",
            "nickname",
            "desc"
        ]
}
```

返回：

```
{
    "took": 10,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 2,
            "relation": "eq"
        },
        "max_score": 1.8419956,
        "hits": [
            {
                "_index": "search_demo",
                "_type": "_doc",
                "_id": "1004",
                "_score": 1.8419956,
                "_source": {
                    "nickname": "红帽子",
                    "id": 1004,
                    "desc": "好的系统必须拥有稳定的系统结构"
                }
            },
            {
                "_index": "search_demo",
                "_type": "_doc",
                "_id": "1005",
                "_score": 1.8419956,
                "_source": {
                    "nickname": "switch游戏机",
                    "id": 1005,
                    "desc": "好的游戏，才会有人购买，比如塞尔达"
                }
            }
        ]
    }
}
```

match_phrase：分词结果必须在text字段内容中都包含而且顺序必须相同，而且必须是连续的（搜索比较严格）

- slop：允许词语间跳过的数量

  ```
  {
      "query": {
          "match_phrase": {
              "desc": {
                  "query": "皮特 姓氏",
                  "slop": 1
              }
          }
      },
          "_source": [
              "id",
              "nickname",
              "desc"
          ]
  }
  ```

  ```
  {
      "took": 14,
      "timed_out": false,
      "_shards": {
          "total": 1,
          "successful": 1,
          "skipped": 0,
          "failed": 0
      },
      "hits": {
          "total": {
              "value": 1,
              "relation": "eq"
          },
          "max_score": 4.7110896,
          "hits": [
              {
                  "_index": "search_demo",
                  "_type": "_doc",
                  "_id": "1011",
                  "_score": 4.7110896,
                  "_source": {
                      "nickname": "皮特",
                      "id": 1011,
                      "desc": "皮特的姓氏好像是彼得"
                  }
              }
          ]
      }
  }
  ```

  

#### 搜索开放了80端口的ssh协议

```
{
    "query" : {
    "bool" : {
    "must" : [//“must”必须，相当于并且AND条件；“should”应该，相当于或者OR条件
    {
        "match":{
            "protocol" : "ssh"
        }
    },
    {
        "match":{
            "port":"80"
        }
    }      
    ]
    }
    }
}
```

#### 搜索开放了https协议或域名包含https的资产

```
{
    "query" : {
    "bool" : {
    "should" : [//“must”必须，相当于并且AND条件；“should”应该，相当于或者OR条件
    {
        "match":{
            "protocol" : "https"
        }
    },
    {
        "match":{
            "ip":"https"
        }
    }      
    ]
    }
    }
}
```



#### 在第2条基础上增加更新时间为2020-07-01之后

```
{
    "query" : {
    "bool" : {
    "should" : [//“must”必须，相当于并且AND条件；“should”应该，相当于或者OR条件
        {
            "bool":{
                "must" : [
                    {
                       "match":{
                           "ip":"https"
                       }
                    },
                    {
                        "range":{
                            "time":{
                                "gt":"2020-01-01"
                            }
                        }
                    }
                ]
            }
        },
        {
            "bool":{
                "must" : [
                    {
                       "match":{
                           "protocol":"https"
                       }
                    },
                    {
                        "range":{
                            "time":{
                                "gt":"2020-01-01"
                            }
                        }
                    }
                ]
            }
        }
    ]
    }
    }
}
```



#### 在第3条基础上增加端口不等于443的

```
{
    "aggs":{
        "group_by_port":{
                "terms":{
                    "field":"port",
                    "size" : 15
            }
        },
        "aggs_port":{
            "terms":{
                "field":"protocol",
                "size":15
            }
        }
    },
    
    "query" : {
    "bool" : {
    "should" : [//“must”必须，相当于并且AND条件；“should”应该，相当于或者OR条件
        {
            "bool":{
                "must" : [
                    {
                       "match":{
                           "ip":"https"
                       }
                    },
                    {
                        "range":{
                            "time":{
                                "gt":"2020-01-01"
                            }
                        }
                    }
                ]
            }
        },
        {
            "bool":{
                "must" : [
                    {
                       "match":{
                           "protocol":"https"
                       }
                    },
                    {
                        "range":{
                            "time":{
                                "gt":"2020-01-01"
                            }
                        }
                    }
                ]
            }
        }
    ],
    "must_not":[
        {
            "match" : {
                "port":"443"
            }
        }
    ]
    }
    }
}
```

#### 端口聚合、按协议和端口聚合，返回前15条

```
{
    "aggs":{
        "group_by_port":{
                "terms":{
                    "field":"port",
                    "size" : 15
            }
        }
        },
        "aggs_port":{
            "terms":{
                "field":"protocol",
                "size":15
            }
        }
    },
    
    "query" : {
    "bool" : {
    "should" : [//“must”必须，相当于并且AND条件；“should”应该，相当于或者OR条件
        {
            "bool":{
                "must" : [
                    {
                       "match":{
                           "ip":"https"
                       }
                    },
                    {
                        "range":{
                            "time":{
                                "gt":"2020-01-01"
                            }
                        }
                    }
                ]
            }
        },
        {
            "bool":{
                "must" : [
                    {
                       "match":{
                           "protocol":"https"
                       }
                    },
                    {
                        "range":{
                            "time":{
                                "gt":"2020-01-01"
                            }
                        }
                    }
                ]
            }
        }
    ],
    "must_not":[
        {
            "match" : {
                "port":"443"
            }
        }
    ]
    }
    }
}
```

显示字段：

```
"aggs":{
                "aggs_port":{
                    "top_hits":{
                        "_source":["ip"]
                    }
                }
            }
```

注：对聚合字段需要进行配置fielddata的查询时内存，修改mapping文件设置 fielddata=true

```
 "properties": {
    "address": {
      "type": "text",
      "fielddata": true
    }
  }
```

#### elasticsearch使用elasticdump导出数据

```
elasticdump --input=http://192.168.17.129:9200/fofa --output=E:\jsonData\test.json --type=data --searchBody {\"aggs\":{\"group_by_port\":{\"terms\":{\"field\":\"port\",\"size\":15}},\"aggs_port\":{\"terms\":{\"field\":\"protocol\",\"size\":15}}},\"query\":{\"bool\":{\"should\":[{\"bool\":{\"must\":[{\"match\":{\"ip\":\"https\"}},{\"range\":{\"time\":{\"gt\":\"2020-01-01\"}}}]}},{\"bool\":{\"must\":[{\"match\":{\"protocol\":\"https\"}},{\"range\":{\"time\":{\"gt\":\"2020-01-01\"}}}]}}],\"must_not\":[{\"match\":{\"port\":\"443\"}}]}}}
```

> 注：需要对查询语句进行转义操作 否则出现json格式错误 需要严格遵循规则

[json格式转换工具在线]: https://www.sojson.com/yasuo.html
