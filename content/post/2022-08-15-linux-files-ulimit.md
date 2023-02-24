---
categories:
- Linux
date: "2022-08-15T00:00:00Z"
description: Linux
keywords: Linux
title: Linux 文件打开数限制
---

 Linux 文件默认打开数限制主要是/etc/security/limits.conf

<!--more-->

```shell
root@racknerd-77489e:~# more /etc/security/limits.conf 
*               soft    nofile           1000000
*               hard    nofile          1000000
```

limits.conf 文件的基本语法是

```
  <域><类型><项目><值>
```

[官方手册](https://linux.die.net/man/5/limits.conf)

查看进程的文件打开数限制

```shell
more /proc/进程号/limits
root@racknerd-77489e:/proc/1145# more limits 
Limit                     Soft Limit           Hard Limit           Units     
Max cpu time              unlimited            unlimited            seconds   
Max file size             unlimited            unlimited            bytes     
Max data size             unlimited            unlimited            bytes     
Max stack size            8388608              unlimited            bytes     
Max core file size        unlimited            unlimited            bytes     
Max resident set          unlimited            unlimited            bytes     
Max processes             unlimited            unlimited            processes 
Max open files            1048576              1048576              files     
Max locked memory         65536                65536                bytes     
Max address space         unlimited            unlimited            bytes     
Max file locks            unlimited            unlimited            locks     
Max pending signals       3860                 3860                 signals   
Max msgqueue size         819200               819200               bytes     
Max nice priority         0                    0                    
Max realtime priority     0                    0                    
Max realtime timeout      unlimited            unlimited            us  
```

修改进程打开文件数限制

```shell
prlimit --pid 进程号 --nofile=65535:65535 #65535为打开限制
prlimit --pid 86672 --nofile=65535:65535
```

