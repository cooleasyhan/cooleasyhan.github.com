Date: 2015-11-09
Title: Kibana elasticsearch logstash
Tags: Linux,Kibana,elasticsearch,logstash,Log
sulg: logstash

##Kibana elasticsearch logstash 日志系统搭建

### 安装
- yum install ruby rubygems
- /usr/bin/gem install bundler 
- yum install java-1.7.0-openjdk
- yum install redis.x86_64   （存储日志信息）
- 启动redis: redis-server /etc/redis.conf
- Kibana elasticsearch logstash 下载
	- https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-1.7.3.zip
	- https://download.elastic.co/kibana/kibana/kibana-4.1.2-linux-x64.tar.gz
	- https://download.elastic.co/logstash/logstash/logstash-1.5.5.zip


### 生成logstash 测试配置文件 stdin.conf
```
input {
stdin {}
}
 
output { 
    elasticsearch { 
 		embedded => true 
	}
} 

```

### 运行logstash
/path_to_logstash/bin/logstash -f stdin.conf
http://logstashhost:9200
显示下面结果则为正常
```
{

    status: 200,
    name: "Blue Marvel",
    cluster_name: "elasticsearch",
    version: {
        number: "1.7.0",
        build_hash: "929b9739cae115e73c346cb5f9a6f24ba735a743",
        build_timestamp: "2015-07-16T14:31:07Z",
        build_snapshot: false,
        lucene_version: "4.10.4"
    },
    tagline: "You Know, for Search"
}
```

在当前窗口输入相关数据用作测试

### 运行kibana
/path_to_kibana/bin/kibana

http://kibanahost:5601/ 

选择Index Patterns


## logstash 配置文件

###Use Case 1
使用外部elasticsearch需要默认logstash模板，可以通过下面方式设置，也可以在logstash上设置
```
curl -XPUT localhost:9200/_template/logstash -d '{
    
        "order":0,
        "template":"logstash-*",
        "settings":{
            "index.refresh_interval":"5s"
        },
        "mappings":{
            "_default_":{
                "dynamic_templates":[
                    {
                        "message_field":{
                            "mapping":{
                                "index":"analyzed",
                                "omit_norms":true,
                                "type":"string"
                            },
                            "match_mapping_type":"string",
                            "match":"message"
                        }
                    },
                    {
                        "string_fields":{
                            "mapping":{
                                "index":"analyzed",
                                "omit_norms":true,
                                "type":"string",
                                "fields":{
                                    "raw":{
                                        "index":"not_analyzed",
                                        "ignore_above":256,
                                        "type":"string"
                                        }
                                    }
                                },
                            "match_mapping_type":"string",
                            "match":"*"
                        }
                    }
                ],
                "properties":{
                    "geoip":{
                        "dynamic":true,
                        "properties":{
                            "location":{
                                "type":"geo_point"
                                }
                            },
                            "type":"object"
                        },
                        "@version":{
                            "index":"not_analyzed",
                            "type":"string"
                            }
                        },
                        "_all":{
                            "enabled":true,
                            "omit_norms":true
                        }
                    }
                },
                "aliases":{}
            }'
```
###Use Case 2

- 错误日志通过 redis 中转
收集所有非INFO的日志，发送至
```
input { 
    file { 
    path => "/home/yihan/access.log" 
    } 
} 
filter { 
if [message] =~ /INFO/ {
     drop{}
}

} 
output { 
      redis { 
      host => 'xx.xx.xx.xx' 
      data_type => 'list' 
      key => 'logstash' 
       } 
} 

```

- 从redis获取日志发送到elasticsearch
```
input {
redis {
                        host => "localhost"
                        type => "redis-input"
                        data_type => "list"
                        key => "logstash"
                }
}  

output {
    elasticsearch {
    host=>"10.115.132.149:9200"
     protocol => http
  }


}

```

