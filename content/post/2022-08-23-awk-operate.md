---
categories: 
- Linux
date: "2022-08-23T00:00:00Z"
description: Linux,awk
keywords: Linux,awk
title: awk 批量操作文件
---

有些个性化需求记录一下，比如批量修改文件

<!--more-->

```shell
[root@localhost demo]# ls
[root@localhost demo]# mkdir -p {1..10}@abc.com
[root@localhost demo]# ls
10@abc.com  1@abc.com  2@abc.com  3@abc.com  4@abc.com  5@abc.com  6@abc.com  7@abc.com  8@abc.com  9@abc.com
[root@localhost demo]# ls -ld * 
drwxr-xr-x 2 root root 6 8月  23 14:09 10@abc.com
drwxr-xr-x 2 root root 6 8月  23 14:09 1@abc.com
drwxr-xr-x 2 root root 6 8月  23 14:09 2@abc.com
drwxr-xr-x 2 root root 6 8月  23 14:09 3@abc.com
drwxr-xr-x 2 root root 6 8月  23 14:09 4@abc.com
drwxr-xr-x 2 root root 6 8月  23 14:09 5@abc.com
drwxr-xr-x 2 root root 6 8月  23 14:09 6@abc.com
drwxr-xr-x 2 root root 6 8月  23 14:09 7@abc.com
drwxr-xr-x 2 root root 6 8月  23 14:09 8@abc.com
drwxr-xr-x 2 root root 6 8月  23 14:09 9@abc.com
[root@localhost demo]# ls -ld * |awk '{print $9}'|xargs -i mv {} {}_bak
[root@localhost demo]# ls
10@abc.com_bak  1@abc.com_bak  2@abc.com_bak  3@abc.com_bak  4@abc.com_bak  5@abc.com_bak  6@abc.com_bak  7@abc.com_bak  8@abc.com_bak  9@abc.com_bak
```

批量删除

```shell
[root@localhost demo]# ls -ld * |awk '{print $9}'|xargs -i rm -rf {}
[root@localhost demo]# ls
```

```shell
awk '{print $9}' #以空格为分隔符 第9列
xargs -i #用于将分段分批传递给其后的{}进行输出，分段会替换{}所在的位置进行输出
```

