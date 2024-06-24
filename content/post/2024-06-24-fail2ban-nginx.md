---
categories:
- Fail2ban
date: "2024-06-17T00:00:00Z"
description: Fail2ban
keywords: Docker Linux Fail2ban Nginx
title: Fail2ban根据Nginx日志封禁IP
---
Fail2ban根据Nginx日志封禁IP
<!--more-->
1:安装fail2ban
```shell
apt install fail2ban
```
2:配置nginx日志
```yaml

log_format json escape=json '{'
                 '"RemoteIP": "$remote_addr", '
                 '"RequestTime": "$time_local", '
                 '"Request": "$request", '
                 '"Status": "$status", '
                 '"BytesSent": "$body_bytes_sent", '
                 '"Referer": "$http_referer", '
                 '"UserAgent": "$http_user_agent", '
                 '"XForwardedFor": "$http_x_forwarded_for", '
                 '"Host": "$host", '
                 '"RequestTime": "$request_time", '
                 '"UpstreamResponseTime": "$upstream_response_time", '
                 '"UpstreamAddr": "$upstream_addr", '
                 '"RequestLength": "$request_length", '
                 '"UpstreamStatus": "$upstream_status", '
                 '"SSLProtocol": "$ssl_protocol"'
                 '}';
    access_log  /var/log/nginx/access.log  json;
```
3:新增fail2ban nginx过滤器
```shell
在这个目录下/etc/fail2ban/filter.d 编辑新建nginx-custom.conf
内容如下:
[Definition]
failregex = ^.*"RemoteIP": "<HOST>".*"Status": "[^23]\d{2}".*$
            ^.*"RemoteIP": "<HOST>".*"Request": "(?:GET|POST|HEAD) (?:/(?:wp-login\.php|wp-admin|administrator|admin|backend|cfg|config)|\.(php|asp|aspx|jsp|cgi)).*".*$
datepattern = "RequestTime"\s*:\s*"%%Y-%%m-%%d %%H:%%M:%%S"
```
4.持久化fail2ban 配置修改/etc/fail2ban/action.d/iptables-persistent.conf
```
[Definition]
actionstart = 
actionstop = 
actioncheck = 
actionban = iptables -I fail2ban-<name> 1 -s <ip> -j DROP
             iptables-save > /etc/iptables/rules.v4
actionunban = iptables -D fail2ban-<name> -s <ip> -j DROP
              iptables-save > /etc/iptables/rules.v4
```
5.配置nginx封禁，配置/etc/fail2ban/jail.d/nginx-custom-jail.conf
```
[nginx-custom-jail]
enabled = true
filter = nginx-custom
logpath = /opt/docker/nginx/log/access.log
maxretry = 3
findtime = 3600
bantime = 1209600
action = iptables-multiport[name=nginx, port="http,https"]
backend = pyinotify
```
6.重启fail2ban
```
systemctl restart fail2ban
查看日志
tail -f /var/log/fail2ban.log
查看封禁
fail2ban-client status nginx-custom-jail
```