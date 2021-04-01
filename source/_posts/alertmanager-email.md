---
title: Alertmanager 配置邮件模板
copyright: true
date: 2019-01-08 23:46:39
tags: Alertmanager
categories: Alertmanager
---


# Alertmanager 配置邮件模板

## Alertmanager 配置

alertmanager 的配置主要是要配置邮箱（通知方式）和模板地址；文档地址：[Alertmanager 地址](https://prometheusAio/docs/alerting/configuration/#email_config)，配置完之后就只需要在模板中定义就好了。

<!-- more -->

```yaml

....
# Whether or not to notify about resolved alerts.
[ send_resolved: <boolean> | default = false ]

# The email address to send notifications to.
to: <tmpl_string>

# The sender address.
[ from: <tmpl_string> | default = global.smtp_from ]

# The SMTP host through which emails are sent.
[ smarthost: <string> | default = global.smtp_smarthost ]

# The hostname to identify to the SMTP server.
[ hello: <string> | default = global.smtp_hello ]

# SMTP authentication information.
[ auth_username: <string> | default = global.smtp_auth_username ]
[ auth_password: <secret> | default = global.smtp_auth_password ]
[ auth_secret: <secret> | default = global.smtp_auth_secret ]
[ auth_identity: <string> | default = global.smtp_auth_identity ]

templates:
- '/etc/alertmanager/templates/xxx.tmpl'

```

## 模板配置

下面给出一份模板配置的文件：

```

{{ define "email.common.html" }}

<div>this is test....</div>

            {{ .Alerts | len }} alert{{ if gt (len .Alerts) 1 }}s{{ end }} for {{ range .GroupLabels.SortedPairs }}
                {{ .Name }}={{ .Value }}
              {{ end }}


在Alertmanager中查看 ：<a href="{{ template "__alertmanagerURL" . }}">View in {{ template "__alertmanager" . }}</a>


                {{ if gt (len .Alerts.Firing) 0 }}
                <tr>
                    <strong>[{{ .Alerts.Firing | len }}] Firing</strong>
                </tr>
                {{ end }}


                 {{ range .Alerts.Firing }}
                <tr style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;">
                  <td style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; vertical-align: top; margin: 0; padding: 0 0 20px;" valign="top">
                    <strong>Labels</strong>

                    {{ range .Labels.SortedPairs }}{{ .Name }} = {{ .Value }}{{ end }}

                    {{ if gt (len .Annotations) 0 }}
                        <strong>Annotations</strong>
                        <br>
                    {{ end }}
                    
                    {{ range .Annotations.SortedPairs }}
                        {{ .Name }} = {{ .Value }}
                    {{ end }}

                    <a href="{{ .GeneratorURL }}" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; color: #348eda; text-decoration: underline; margin: 0;">
                        Source
                    </a>
                        
                    <br style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;" />
                  </td>
                </tr>
                {{ end }}



<p>
<p>
<p>


                {{ if gt (len .Alerts.Resolved) 0 }}
                  {{ if gt (len .Alerts.Firing) 0 }}
                  {{ end }}

                  <td>
                    <strong>[{{ .Alerts.Resolved | len }}] Resolved</strong>
                  </td>
                {{ end }}



                {{ range .Alerts.Resolved }}
                <tr>
                  <td>
                    <strong>Labels</strong>
                    <br/>
                    {{ range .Labels.SortedPairs }}
                        {{ .Name }} = {{ .Value }}
                    {{ end }}
                    
                    {{ if gt (len .Annotations) 0 }}
                        <strong>Annotations</strong>
                    {{ end }}

                    {{ range .Annotations.SortedPairs }}
                        {{ .Name }} = {{ .Value }}
                    {{ end }}

                    <a href="{{ .GeneratorURL }}" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; color: #348eda; text-decoration: underline; margin: 0;">
                        Source
                    </a>
                    <br style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;" />
                  </td>
                {{ end }}



{{end}}

```
 
这里要注意的是第一行:` define "email.common.html" `记住结尾一定要有 `end` 与之对应，因为 golang 的 template 模板限制。
其实这个 tmpl 文件就是 golang 的 template 模板。以前看到还有点懵，用过一次 golang 中 template 功能之后，会有很多明白的地方。

在一个文件中也是可以定义多个模板的只需要有多个` define "xxx" `即可。记住在 alertmanager 的配置文件`alertmanager.yml`中一定要有

```yaml
html:  {{template "email.common.html" }} .
``` 

这里的`email.common.html`要与tmpl文件中定义的相同。

源码中有示例: [alertmanager template](https://github.com/prometheus/alertmanager/blob/master/template/default.tmpl)

而tmpl文件里面的内容都在这个go文件中 [template.go](https://github.com/prometheus/alertmanager/blob/master/template/template.go) 可以看到里面有个`Data`struct。这里就是它的详细信息了。

## 扩展阅读

[package text/template](https://godoc.org/text/template)

[package html/template](https://golang.org/pkg/html/template/)

[模板处理](https://astaxie.gitbooks.io/build-web-application-with-golang/zh/07.4.html)

[golang 模板(template)的常用基本语法](https://www.kancloud.cn/cserli/golang/531904)
