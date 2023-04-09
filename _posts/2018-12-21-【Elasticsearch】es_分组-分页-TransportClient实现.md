---
layout: post
title:  "Elasticsearch | es_分组-分页-TransportClient实现"
date:   2018-12-21 17:05:42
categories: 大数据
tags: ES Java
mathjax: true
---

* content
{:toc}

使用TransportClient实现java常用操作ES接口




说在前面：

Elasticsearch Java API 有四种实现方式：分别是 TransportClient、RestClient、Jest、Spring Data Elasticsearch。
本文使用第一种方式，也就是 TransportClient 的方式进行实现。想要了解其他三种的方式可以看一下这篇文章：https://blog.csdn.net/qq_33314107/article/details/80725913

# 总结：

1、group 之后**不能自动分页**，需要手动设置；

2、size 需要指定，否则会出错。

---

## group 之后分页、排序实现

需要手动截取分页对应范围内的数据。

比如：这里

倒序，获取到数据集的第 (currentPage-1) * limit 到 currentPage * limit 条记录；

升序，获取到数据集的 第 buckets.size() - (currentPage - 1) * limit 到 buckets.size() - currentPage * limit 条记录。

    // 获取到 response 之后
    Aggregations aggregations = response.getAggregations();
    Terms carids =  aggregations.get("group_car_bayId");
    List<? extends Terms.Bucket> buckets =  carids.getBuckets();
    List<carBean> listCarTgs = new ArrayList<>();
    carBean carBean ;
    // buckets 全部数据，分页就是取固定位置的 limit 条数据
    // 默认按照统计之后的数量倒序， 如果要正序，则第一页从最后一条开始取
    if(buckets.size()>0) {
        int i=0;
        if("desc".equalsIgnoreCase(order)) {// 倒序
            for(Terms.Bucket bucket : buckets){
                if(i++<(currentPage-1) * limit){
                    continue;
                }
                if (i > currentPage * limit) {
                    break;
                }
                carBean = new carBean();
                carBean.setPassTimes((int)bucket.getDocCount());
                carBean.setBayId(bucket.getKeyAsString().split("_")[2]);
                carBean.setPlateNumber(bucket.getKeyAsString().split("_")[0]);
                carBean.setPlateType(bucket.getKeyAsString().split("_")[1]);
                listCarTgs.add(carBean);
            }
        }else if("asc".equalsIgnoreCase(order)) {// 升序
            for(i = buckets.size() - 1; i >= 0; i--){
                if(i < buckets.size() - currentPage * limit){
                    break;
                }
                if(i > buckets.size() - (currentPage - 1) * limit) continue;;
                carBean = new carBean();
                carBean.setPassTimes((int)buckets.get(i).getDocCount());
                carBean.setBayId(buckets.get(i).getKeyAsString().split("_")[2]);
                carBean.setPlateNumber(buckets.get(i).getKeyAsString().split("_")[0]);
                carBean.setPlateType(buckets.get(i).getKeyAsString().split("_")[1]);
                listCarTgs.add(carBean);
            }
        }
    }

## 单个 group

注意：需要设置 size 

    TermsAggregationBuilder tb=  AggregationBuilders.terms("group_bayId").field("bay_id").size(Integer.MAX_VALUE);



## 多个 group

以脚本的形式

    TermsAggregationBuilder tb=  AggregationBuilders.terms("group_carId").script(new Script("doc['plateNumber'].value+'_'+doc['plateType'].value"));
                tb.subAggregation(AggregationBuilders.topHits("max_time").sort("reportTime", SortOrder.DESC).size(1));
                
再比如：三个 group

    BoolQueryBuilder filter = QueryBuilders.boolQuery();
    if (carList != null && carList.size() >0) {
        filter.must(QueryBuilders.termsQuery("car_plate_number", carList.stream().map(SimpleCar:: getPlateNumber).collect(Collectors.toList())));
    }
    if (startTime != null && endTime != null) {
        filter.filter(QueryBuilders.rangeQuery("timestamp").gt(startTime.getTime()).lt(endTime.getTime()));
    } else if (startTime != null) {
        filter.filter(QueryBuilders.rangeQuery("timestamp").gt(startTime.getTime()));
    } else if (endTime != null) {
        filter.filter(QueryBuilders.rangeQuery("timestamp").lt(endTime.getTime()));
    }
    FieldSortBuilder sort = SortBuilders.fieldSort("transit_times").order("asc".equalsIgnoreCase(order)?SortOrder.ASC:SortOrder.DESC);
    TermsAggregationBuilder termsAggregationBuilder =  AggregationBuilders.terms("group_car_bayId")
            .script(new Script("doc['car_plate_number'].value+'_'+doc['car_plate_type'].value + '_' +doc['bay_id'].value")).size(Integer.MAX_VALUE);
    SearchResponse response = search(filter, sort, termsAggregationBuilder, elasticsearchProperties.getTgsIndex(), elasticsearchProperties.getTgsType(),
            (currentPage-1) * limit, 0);
    Aggregations aggregations = response.getAggregations();

                
例子2：

    TermsAggregationBuilder tb=  AggregationBuilders.terms("group_bayId").field("bay_id").size(Integer.MAX_VALUE);
        tb.order(BucketOrder.count(false));
        
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20181221170450368.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1ByZXBhcmVk,size_16,color_FFFFFF,t_70)     
        

        
