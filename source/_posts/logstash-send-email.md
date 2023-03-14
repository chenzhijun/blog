---
title: 使用Logstash发送异常邮件
date: 2018-01-09 14:01:10
tags:
    - ELK
    - Logstash
categories: ELK
---

# 使用Logstash发送异常邮件

前端时间我们讲了[如何使用elk搭建日志系统](http://chenzhijun.top/2017/12/12/elasticsearch-logstash-kibana-part/),以及如何[使用Docker搭建ELK日志系统](http://chenzhijun.top/2017/12/27/elk-docker/)。虽然我们可以不用再去日志服务器找日志了，但是这样也有问题，我怎么知道什么时候会出现异常，不出现异常我也没必要去kibana查日志啊。

今天我们就要解决这个问题。当然解决的方式比较简单。如果有大神有更好的方式欢迎一起分享。

## 使用Logstash发送邮件

我们使用的是Logstash来发送邮件，网上我也搜了elastalert,但是感觉又多了一个服务，又要多去维护一个服务。后来发现logstash自带了邮件发送功能，那就直接用logstash就好了。

非常的简单易用，在`logstash.conf`中增加如下配置：

```conf
input {
  beats {
    host => "localhost"
    port => "5043"
  }
}

filter {
   if [fields][doc_type] == 'order' {
    grok {
			match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{JAVALOGMESSAGE:msg}" }
		}
   }
}

output {
  stdout { codec => rubydebug }
  elasticsearch {
        hosts => ["localhost:9200"]
        index => "%{[fields][doc_type]}-%{+YYYY.MM.dd}"
    }

  if "ERROR" == [level] {
    email {
        to => "5228******@qq.com,chen****@163.cn"
        cc => "email_chen****@163.com"
        via => "smtp"
        subject => "标题，ERROR: %{[fields][doc_type]}项目出现异常"
        htmlbody => "消息主体：%{message}"
        body => "Tags: %{tags}\\n\\Content:\\n%{message}"
        from => "email_chen****@163.com"
        address => "smtp.163.com"
        username => "email_chen****@163.com"
        password => "*****" # pop3密码或者登陆密码
    }
  }
}
```

主要是增加了下面这段

```conf
  if "ERROR" == [level] {
    email {
        to => "5228******@qq.com,chen****@163.cn"
        cc => "email_chen****@163.com"
        via => "smtp"
        subject => "标题，ERROR: %{[fields][doc_type]}项目出现异常"
        htmlbody => "消息主体：%{message}"
        body => "Tags: %{tags}\\n\\Content:\\n%{message}"
        from => "email_chen****@163.com"
        address => "smtp.163.com"
        username => "email_chen****@163.com"
        password => "*****" # pop3密码或者登陆密码
    }
  }
```

我们只对ERROR级别的日志进行发送邮件，这里用了if条件语句。如果你看过之前的两篇文章，我想这里你是很容易就能弄懂的。当然这种方式不一定很好，如果你有更好的想法，欢迎交流。

<!--

```conf
input {
  beats {
    host => "localhost"
    port => "5044"
  }
}

filter {
    grok {
			match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{JAVALOGMESSAGE:msg}" }
		}
}

output {
  stdout { codec => rubydebug }

  if "ERROR" == [level] or [message] =~ /xception/ {
    email {
        to => "xukai@ap-ec.cn,wangfei@ap-ec.cn,xiaoxq@ap-ec.cn,zhangchi@ap-ec.cn,shangxw@ap-ec.cn,guisw@ap-ec.cn,chenzj@ap-ec.cn"
        #to => "chenzj@ap-ec.cn"
        cc => "noreply_czj@163.com"
        via => "smtp"
        subject => "请注意项目可能出现异常,异常文件:%{source}"
        # htmlbody => "消息主体：%{message}"
        body => "日志信息:\n%{message}\n\n请在日志文件中查看详细信息,机器地址：%{[fields][server_ip]}。\n\n\n\n\n-----------------------------华丽的分割线---------------------\n这是系统发送邮件，请勿回复。\n如有需要请联系chenzhijun。chenzj@ap-ec.cn"
        from => "noreply_czj@163.com"
        address => "smtp.163.com"
        username => "noreply_czj@163.com"
        password => "noreplyczj123"
    }
    http {
        url => "https://oapi.dingtalk.com/robot/send?access_token=3f39f66edc27dbd5c8206183741c372ec459b653af0cd90841edffbbb872cf1b"
        content_type => "application/json"
        http_method => "post"
        message => '{"msgtype": "text","text": {"content": "%{message}"}}'
    }
  }
}
```

```yml
filebeat.prospectors:

# Each - is a prospector. Most options can be set at the prospector level, so
# you can use different prospectors for various configurations.
# Below are the prospector specific configurations.

- type: log

  # Change to true to enable this prospector configuration.
  enabled: true

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /home/cncsen/*/*.log
    #- /home/cncsen/customer/*/*.log
    #- /home/cncsen/contract/*.log
    #- c:\programdata\elasticsearch\logs\*
  multiline:
      pattern: ^\d{4}
      negate: true
      match: after
  fields:
    server_ip: 221

```
-->
