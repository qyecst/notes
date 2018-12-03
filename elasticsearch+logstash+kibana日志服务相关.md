<!--
{
    "title": "elasticsearch+logstash+kibana日志服务相关",
    "create": "2018-12-03 15:36:09",
    "modify": "2018-12-03 15:36:09",
    "tag": [
        "elk",
        "elasticsearch",
        "logstash",
        "kibana"
    ],
    "info": [
        "主机部署//todo"
    ]
}
-->

## elasticsearch配置

环境配置：

```conf
# elasticsearch默认不允许root执行
useradd es_user

# /etc/security/limits.conf 调整文件句柄 `ulimit -Hn`
es_user  hard  nofile  65536
es_user  soft  nofile  65536

# /etc/sysctl.conf 调整内存映射 `sysctl -p`
vm.max_map_count=655360
```

配置文件：

```conf
# es_home/bin/elasticsearch

# 调整使用内存
ES_JAVA_OPTS="-Xms1g -Xmx1g"
```

## logstash配置文件

```conf
# logstash -f /etc/logstash.conf

# 输入配置，tcp/udp/file/.etc
input {
    udp {
        port => "9652"
        type => "rsys"
    }
    tcp {
        port => "9652"
        type => "rsys"
    }
    udp {
        port => "9651"
        type => "hw"
    }
    tcp {
        port => "9651"
        type => "hw"
    }
    udp {
        port => "9650"
        type => "h3c"
    }
    tcp {
        port => "9650"
        type => "h3c"
    }
}
# 过滤配置
filter {
    if [type] == "rsys" {
        grok {
            # 匹配规则
            match => {
                "message" => "<%{BASE10NUM:syslog_pri}>%{SYSLOGTIMESTAMP:timestamp} (?<hostname>.*?) %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:message}"
            }
            # 移除字段
            remove_field => [ "timestamp" ]
            # 重写message
            overwrite => ["message"]
        }
    }
    else if [type] == "h3c" {
        grok {
            match => {
                "message" => "<%{BASE10NUM:syslog_pri}>%{SYSLOGTIMESTAMP:timestamp} %{YEAR:year} %{DATA:hostname} %%%{DATA:vvmodule}/%{POSINT:severity}/%{DATA:digest}: %{GREEDYDATA:message}"
            }
            remove_field => [ "year" ]
            # 添加字段
            add_field => {
                "severity_code" => "%{severity}"
            }
            overwrite => ["message"]
        }
    }
    else if [type] == "hw" {
        grok {
            match => {
                "message" => "<%{BASE10NUM:syslog_pri}>(?<timestamp>.*? .*?) %{DATA:hostname} %%%{DATA:ddModuleName}/%{POSINT:severity}/%{DATA:Brief}:%{GREEDYDATA:message}"
            }
            remove_field => [ "timestamp" ]
            add_field => {
                "severity_code" => "%{severity}"
            }
            overwrite => ["message"]
        }
    }
    mutate {
        # 正则表达式替换gsub '字段', '匹配式', '值'
        gsub => [
            "severity", "0", "Emergency",
            "severity", "1", "Alert",
            "severity", "2", "Critical",
            "severity", "3", "Error",
            "severity", "4", "Warning",
            "severity", "5", "Notice",
            "severity", "6", "Informational",
            "severity", "7", "Debug"
        ]
        # 类型转换
        convert => {
            "field1" => "string"
            "field2" => "integer"
        }
        # 复制，已存在则重写
        copy => {"field1" => "field2"}
        # 大小写转换lowercase & uppercase
        lowercase => [ "field1" ]
        uppercase => [ "field2" ]
        # 重命名字段
        rename => {"field1" => "field2"}
        # 去除字段前后空格
        strip => ["field1"]
        # 更新字段值
        update => {"field1" => "new_value"}
        # 替换字段值，不存在则添加
        replace => {"field1" => "new_value"}
        # 顺序 rename => update => replace => convert => gsub => uppercase => lowercase => strip => remove => split => join => merge
    }
}
output {
    elasticsearch {
        hosts => [ "elasticsearch:9200" ]
    }
}
```

## docker-compose方式部署

### docker-compose文件

```dockercompose
version: "3.5"
services:
    elasticsearch:
        image: elasticsearch:6.5.0
        restart: always
        #ports:
        #  - "9300:9200"
        environment:
            discovery.type: single-node
            ES_JAVA_OPTS: -Xms512m -Xmx512m # 配置ES的java虚机内存使用
        container_name: elasticsearch
        hostname: elasticsearch
        volumes:
            - ./elasticsearch_data:/usr/share/elasticsearch/data:rw
    logstash:
        image: logstash:6.5.0
        restart: always
        ports:
            - "9652:9652/tcp"
            - "9652:9652/udp"
            - "9650:9650/tcp"
            - "9650:9650/udp"
            - "514:9651/tcp"
            - "514:9651/udp"
        depends_on:
            - elasticsearch
        container_name: logstash
        hostname: logstash
        command: logstash -f /etc/logstash.conf
        volumes:
            - ./logstash.conf:/etc/logstash.conf:ro
    kibana:
        image: kibana:6.5.0
        restart: always
        ports:
            - "5601:5601"
        environment:
            ELASTICSEARCH_URL: http://elasticsearch:9200
        depends_on:
            - elasticsearch
        container_name: kibana
        hostname: kibana
```

### 目录结构

```txt
elk_dir
├── docker-compose.yaml
├── elasticsearch_data
└── logstash.conf
```
