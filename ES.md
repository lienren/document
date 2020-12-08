# Elasticsearch 使用总结

1. 安装ES 7.X版本
2. 安装ik分词（注意与ES版本一致）
> 本文针对 ES7.8版本执行完成

### 相关文档
[ES官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
[ik分词](https://github.com/medcl/elasticsearch-analysis-ik)
[ES7.8破解](https://www.cco.xyz/archives/811)

### 常用命令
```
# 启动
systemctl start elasticsearch.service

# 停止
systemctl stop elasticsearch.service
```

### 新建文档结构
**PUT** http://ip:9200/users/_doc/1
```json
{
    "mappings": {
        "properties": {
            "user_name": {
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_smart"
            },
            "user_phone": {
                "type": "keyword",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_smart"
            },
            "user_age": {
                "type": "integer"
            },
            "user_remark": {
                "type": "text",
                "analyzer": "ik_max_word",
                "search_analyzer": "ik_smart"
            },
            "create_time": {
                "type": "date",
                "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
            }
        }
    }
}
```

### 新建数据
**POST** http://ip:9200/users/_doc/1
```json
{
  "user_name":"XXXXX",
  "user_phone": "186XXXXXXXX",
  "user_age": 30,
  "user_remark": "这里省略一万字",
  "create_time": "2020-12-07 12:59:59"
}
```

### 查询
**GET** http://10.10.151.52:9200/users/_search
```json
{
    "from": 0,
    "size": 10,
    "query": {
        "match": {
            "user_name": "李"
        }
    },
    "highlight": {
        "pre_tags": [
            "<tag1>"
        ],
        "post_tags": [
            "</tag1>"
        ],
        "fields": {
            "user_name": {}
        }
    }
}
```

### 分词分析
**GET** http://ip:9200/_analyze?pretty
```json
{
    "analyzer": "ik_smart",
    "text": "一万个字"
}
```

### 删除表
**DELETE** http://ip:9200/users
