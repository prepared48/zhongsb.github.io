---
layout: post
title:  "Elasticsearch | ES常用操作--postman实现"
date:   2019-03-26 08:00:43 
categories: 大数据
tags: ES
mathjax: true
---

* content
{:toc}

ES 增删改查





# ES常用操作--postman实现
## es-Range Aggregation(范围聚合)

A multi-bucket value source based aggregation that enables the user to define a set of ranges－每个代表一个bucket。

在聚合过程中，从每个文件中提取的值将根据每个存储区范围进行检查，并对相关/匹配文档进行“bucket”，值得注意的是，注意，**此聚合包含了from值，不包括每个范围的to的值**。

例子：


    {
        "aggs" : {
            "price_ranges" : {
                "range" : {
                    "field" : "price",
                    "ranges" : [
                        { "to" : 50 },
                        { "from" : 50, "to" : 100 },
                        { "from" : 100 }
                    ]
                }
            }
        }
    }

响应结果：
    
    {
        ...
     
        "aggregations": {
            "price_ranges" : {
                "buckets": [
                    {
                        "to": 50,
                        "doc_count": 2
                    },
                    {
                        "from": 50,
                        "to": 100,
                        "doc_count": 4
                    },
                    {
                        "from": 100,
                        "doc_count": 4
                    }
                ]
            }
        }
    }




http://cwiki.apachecn.org/pages/viewpage.action?pageId=10028848

## es建表

发送 put 请求

    192.168.2.11:9200/IndexName

文本raw，数据为json格式

    {
        "settings":{
            "number_of_shards":5,
            "number_of_replicas":1
        },
        "mappings":{
            "TypeName":{
                "dynamic":"strict",
                "properties":{
                    "tableId":{"type":"string","store":"yes","index":"not_analyzed"},
                    "title":{"type":"string","store":"yes","index":"analyzed","analyzer": "ik_max_word","search_analyzer": "ik_max_word"},
                    "author":{"type":"string","store":"yes","index":"analyzed","analyzer": "ik_max_word","search_analyzer": "ik_max_word"},
                    "summary":{"type":"string","store":"yes","index":"analyzed","analyzer": "ik_max_word","search_analyzer": "ik_max_word"},
                    "contextSrc":{"type":"string","store":"yes","index":"not_analyzed","ignore_above": 100},
                    "context":{"type":"string","store":"yes","index":"analyzed","analyzer": "ik_max_word","search_analyzer": "ik_max_word"},
                    "keywords":{"type":"string","store":"yes","index":"analyzed","analyzer": "ik_max_word","search_analyzer": "ik_max_word"},
                    "publishDate":{"type":"string","store":"yes","index":"not_analyzed"},
                    "createTime":{"type":"string","store":"yes","index":"not_analyzed"},
                    "modifyTime":{"type":"string","store":"yes","index":"not_analyzed"},
                    "deleteTime":{"type":"string","store":"yes","index":"not_analyzed"},
                    "url":{"type":"string","store":"yes","index":"not_analyzed"},
                    "isDeleted":{"type":"string","store":"yes","index":"not_analyzed"}
                }
            }
        }
    }


###  举例说明：

PUT http://ip:port/indexname/

    {
        "settings":{
    		"index":{
    			"number_of_shards":5,
    			"number_of_replicas":1
          }
            
        },
        "mappings":{
            "koms_aibox_currentgpsclose":{
                "properties":{
                    "id":{"type":"keyword"},
                    "transit_time":{"type":"keyword"},
                    "alerm_state":{"type":"keyword"},
                    "gps_time":{"type":"keyword"},
                    "travel_time":{"type":"keyword"},
    				"abnormal_coos":{"type":"keyword"},
    				"date":{"type":"keyword"}
                }
            }
        }
    }
   
   
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190326080436603.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1ByZXBhcmVk,size_16,color_FFFFFF,t_70)



注：


https://ask.hellobi.com/blog/jack/5401

http://www.cnblogs.com/sunny3096/p/7504630.html

https://blog.csdn.net/liuxiao723846/article/details/78444472

## es批量添加数据

示例

    XPOST ip:port/indexname/typename/_bulk

格式说明
    
    post ip：port索引/类型/_bulk


使用 POSTMAN 批量导出数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190326080512132.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1ByZXBhcmVk,size_16,color_FFFFFF,t_70)

## es添加数据

   
   
    POST http://ip:port/indexname/typename/3
    

说明：

3 是 id，可写可不写，不写自动生产



    {
    	"tgs_id":"433100100153",
    	"field1":"value1",
    	"field2":"value2",
    	"field3":"value3",
    	"field4":"value4"
    }
    
pass_car

    {
	    	"tgs_id":"433100100153",
	    	"field1":"value1",
	    	"field2":"value2",
	    	"field3":"value3",
	    	"field4":"value4"
            "time_frame": "9"
    }

    
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9ub3RlLnlvdWRhby5jb20veXdzL3B1YmxpYy9yZXNvdXJjZS8yNmRiNGVjYzdhNTFkYzA2YzBhM2RhN2Y4OGQ1ZDI1MS94bWxub3RlLzk2Q0U3OTk1NDlCMzREMDU5QkMwMTlFRUZCNTY0RUNCLzE4MTk1?x-oss-process=image/format,png)


https://blog.csdn.net/hzrandd/article/details/46909635

https://blog.csdn.net/xsdxs/article/details/72849796

## es 查询数据

XGET http://ip：port/indexname/typename/_search?scroll=10m

BODY

    {
        "query": { "match_all": {}},
        "size":  1000
    }
    

## 其他基础操作

### 查询所有索引

    GET http://ip：port/_cat/indices?v
    
    
### 删除索引

    DELETE /my_index
    
### 删除多个索引

    DELETE /index_one,index_two
    DELETE /index_*
    
### 删除全部索引

    DELETE /_all
    DELETE /*

https://www.elastic.co/guide/cn/elasticsearch/guide/current/_deleting_an_index.html
    
