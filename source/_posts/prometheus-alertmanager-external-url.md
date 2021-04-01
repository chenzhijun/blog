---
title: alertmanager发送的邮件中external-url修改机器名为IP地址
copyright: true
date: 2018-06-09 16:33:37
tags: ['prometheus','alertmanager']
categories: 监控
---

# alertmanager发送的邮件中external-url修改机器名为IP地址

在使用alertmanager发送报警邮件的时候，我们通常会采用模板。比如我的一个模板：
<!--more-->
```tmpl
{{ define "email.czj.html" }}
详情：<br/><br/>


{{ range .Alerts.Firing }}
                <tr>
                  <td class="content-block">
                    {{ if gt (len .Annotations) 0 }}<strong>描叙信息：</strong><br />{{ end }}
                    {{ range .Annotations.SortedPairs }}{{ .Name }} = {{ .Value }}<br />{{ end }}
                    <strong>指标详情：</strong><br />
                    {{ range .Labels.SortedPairs }}{{ .Name }} = {{ .Value }}<br />{{ end }}
                    
                    <a href="{{ .GeneratorURL }}">链接到Prometheus</a><br />
                  </td>
                </tr>
                <br/><br/>
{{ end }}

<br/><br/>
<div>
</div>
<br/>
<div class="footer">
          <table width="100%">
            <tr>
              <td class="aligncenter content-block" style="text-align: center;"><a href='{{ .ExternalURL }}'>由 {{ template "__alertmanager" . }}发送</a></td>
            </tr>
          </table>
        </div>

{{ end }}
```

如果注意到会发现有：GeneratorURL和ExternalURL。这两者默认使用的是机器名称也就是`hostname`。这样我们就很难在邮件中获取到实际的promtheus和alertmanager地址。查了很多资料，最后发现在prometheus和alertmanager启动的时候我们可以设置这两个值的。prometheus的启动命令是：

```shell
./prometheus.exe --web.external-url="http://127.0.0.1:9090/prom" \
                --web.route-prefix=prom \
                --web.listen-address="0.0.0.0:9090"
```

alertmanager启动命令是：

```shell
./alertmanager.exe --config.file=config163.yml \
                    --web.external-url=http://127.0.0.1:9093/alertmanager \              
                    --web.route-prefix=alertmanager
```

这里的`web.external-url`也就是GeneratorURL和ExternalURL两者在email中的指，在设置`web.external-url`的同时我们需要记得设置`web.router-prefix`的值，应为`web.router-prefix`的默认值是`web.external-url`，如果不同时指定`web.router-prefix`那么就将会出现特别神奇的效果，你需要重复输入两个地址才能访问到相应的prometheus和alertmanager，这个参数是指的路径。所以一定要设置`web.router-prefix`,你也可以设置成`--web.route-prefix=""`这样来将子路径就设置为根路径。