<!--
{
    "title": "elk接入交换机&路由器等网络设备日志",
    "create": "2018-12-03 23:37:34",
    "modify": "2018-12-13 23:00:34",
    "tag": [
        "elk",
        "elasticsearch",
        "logstash",
        "kibana",
        "h3c",
        "huawei",
        "cisco",
        "switch",
        "router",
        "ac",
        "ap",
        "rsyslog"
    ],
    "info": []
}
-->

## 网络设备配置

```command
// cisco
dev(config) logging on // 开启log
dev(config) logging <192.168.1.1|log_host_ip> // log主机地址
dev(config) logging trap <6|log_level> //记录日志级别
dev(config) logging facility <local6|fa_level> // facility标识
dev(config) logging source-interface <loopback0|interface> // 发出日志的源ip地址
dev(config) service timestamps log datetime localtime // 日志时间戳设置

// h3c
[dev] info-center enable // 开启info-center
[dev] info-center loghost <192.168.1.1> port 514 facility local6 // 设置日志主机/端口/日志级别
[dev] info-center loghost source <interface> // 设置日志源ip地址
[dev] info-center timestamp loghost date // 设置日志时间戳

// huawei
[dev] info-center enable // 开启info-center
[dev] info-center loghost <192.168.1.1> facility <local6|fa_level> // 设置日志主机/日志级别
[dev] info-center loghost source <interface> // 设置日志源ip地址
[dev] info-center timestamp log date // 设置日志时间戳
```

## logstash配置

```logstash
input {
    udp {
        port => "9652"
        type => "rsys"
    }
    udp {
        # rsyslog使用514/udp，但是监听1000以下端口需要root权限，故而宿主机514转发至docker内高端口，应对不能设置log端口的网络设备
        port => "9651"
        type => "hw"
    }
    udp {
        port => "9650"
        type => "h3c"
    }
}
filter {
    if [type] == "rsys" {
        grok {
            match => {
                "message" => "<%{BASE10NUM:syslog_pri}>%{SYSLOGTIMESTAMP:timestamp} (?<hostname>.*?) %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:message}"
            }
            remove_field => [ "timestamp" ]
            overwrite => ["message"]
        }
    }
    else if [type] == "h3c" {
        grok {
            match => {
                "message" => "<%{BASE10NUM:syslog_pri}>%{SYSLOGTIMESTAMP:timestamp} %{YEAR:year} %{DATA:hostname} %%%{DATA:vvmodule}/%{POSINT:severity}/%{DATA:digest}: %{GREEDYDATA:message}"
            }
            remove_field => [ "year" ]
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
    }
}
output {
    elasticsearch {
        hosts => [ "elasticsearch:9200" ]
    }
}
```

```logstash
if [type] == "cisco"{
    grok{
        match => {
            "message" => "<%{BASE10NUM:syslog_pri}>%{NUMBER:log_sequence}: %{SYSLOGTIMESTAMP:timestamp}: %%{DATA:facility}-%{POSINT:severity}-%{CISCO_REASON:mnemonic}: %{GREEDYDATA:message}"
            }
        add_field => {
            "severity_code" => "%{severity}"
            }
        overwrite => ["message"]
    }
}

# 自动生成grok表达式网站/测试grok表达式网站
# http://grokconstructor.appspot.com/do/match
# https://grokdebug.herokuapp.com/
```

## elk的docker-compose文件

```dockercompose
version: "3.5"
services:
  elasticsearch:
    image: elasticsearch:6.5.0
    restart: always
    #ports:
    #  - "9200:9200"
    environment:
      discovery.type: single-node
      ES_JAVA_OPTS: -Xms512m -Xmx512m
    container_name: elasticsearch
    hostname: elasticsearch
    volumes:
      - ./elasticsearch_data:/usr/share/elasticsearch/data:rw
  logstash:
    image: logstash:6.5.0
    restart: always
    ports:
      - "9652:9652/udp"
      - "9650:9650/udp"
      - "514:9651/udp" # rsyslog使用514/udp，但是监听1000以下端口需要root权限，故而宿主机514转发至docker内高端口，应对不能设置log端口的网络设备
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

## rsyslog配置

```bash
# 默认/etc/rsyslog.conf

*.* @192.168.1.1:9652
*.* @@192.168.1.1:9652
# *.* 表示所有类型日志 @ 表示udp @@ 表示tcp : 后接端口
```
