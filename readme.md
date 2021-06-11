# Spring-boot-elasticsearch-timeseries-demo

该项目作为Elasticsearch用于时序数据库用途的demo，核心是使用ES模板来快速生成分片索引，查询时借助索引别名完成分片的聚合查询，本例按月进行分片，例如my_index_test_2021-6

## 环境
* ES集群(7.1.1)：
    * node1：192.168.56.101：9200
    * node2：192.168.56.102：9200
    * node3：192.168.56.103：9200

## 前提
需要先创建好模板，完成后当插入文档时，若文档的索引名符合模板，则使用模板创建
* template属性：模板匹配，通配符*用于匹配索引
* aliases：别名，用于查询使用
```
curl -XPUT "http://192.168.56.101:9200/_template/my_index_test_template" -d '{
  "template": "my_index_test*",
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 2
  },
  "mappings": {
        "properties": {
            "createTime": {
                "type": "date",
                "format": "yyyy-MM-dd HH:mm:ss || yyyy-MM-dd || yyyy/MM/dd HH:mm:ss|| yyyy/MM/dd ||epoch_millis"
            },
            "id": {
                "type": "keyword"
            },
            "name": {
                "type": "keyword"
            }
        }
    },
  "aliases": {"my_index_test_alias":{}}
}'
```

## POM
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 使用
* 自行修改配置文件中的elasticsearch地址
* 运行DemoEsTimeseriesApplication.java
* 访问http://localhost:8844/insert，插入一条数据，会自动根据当前时间存入相应的分片索引中
* 访问http://localhost:8844/query，查询所有分片
* TODO：可以定期删除旧索引