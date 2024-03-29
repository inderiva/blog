---
categories: 
- linux
date: "2020-05-07T00:00:00Z"
description: linux
keywords: linux
title: awk命令集合
---

Linux处理文本工具：

```
    grep： 过滤文本内容 
	sed：  编辑文本内容 
	awk	   显示文本 
```

<!--more-->

awk:Aho,Kernighan and Weinberger
	报告生成器，以特定的条件查找文本内容，再以特定的格式显示出来


awk命令的格式：

awk [option] 'script' file1 file2 ...

awk [option] 'PATTERN{action}' file1 file2 ...

PATTERN：
	用文本字符与正则表达式元字符描述的条件,可以省略不写
	
action：
	print
	printf 指定输出项的格式；格式必须写

option选项：
	-F	指定文本分割符

awk处理文本机制：
awk将符合PATTERN的文本逐行取出，并按照指定的分割符(默认为空白,
通过-F选项可以指定分割符)进行分割，然后将分割后的每段按照特定的格式输出 



awk的输出：

一、print

print的使用格式：
	print item1,item2,....
	
注意：
1、各项目间使用逗号分隔开，而输出时以空白字符作为分隔
2、输出的item可以为字符串、数值、当前的记录的字段($1)、变量或者awk的表达式；数值会先转换成字符串，然后输出
3、print命令后面的item可以省略，此时其功能相当于print $0($0代表未分割的整行文本内容),因此,如果想输出空白行，则需要使用print "";


以空白分割，显示文本中的第1段及第2段内容

awk '{print $1,$2}' test.txt 

this is

[root@shell ~]# df -hT | sed '1d' | awk '{print "Disk name: ", $1,"Mount Point: ", $7, "Total Size: ", $3, "Free size ", $5}'






awk变量

1、awk内置变量之记录变量

FS： 指定读取文本时，所使用的行分隔符,默认为空白字符；相当于awk的-F选项 
OFS：指定输出的分隔符，默认为空白字符；

[root@localhost ~]# head -n 1 /etc/passwd | awk -F: '{print $1,$7}'
root /bin/bash
[root@localhost ~]# 
[root@localhost ~]# head -n 1 /etc/passwd | awk 'BEGIN{FS=":"}{print $1,$7}'
root /bin/bash

[root@localhost ~]# head -n 1 /etc/passwd | awk -F: '{print $1,$7}'
root /bin/bash
[root@localhost ~]# head -n 1 /etc/passwd | awk -F: 'BEGIN{OFS="---"}{print $1,$7}'
root---/bin/bash
[root@localhost ~]# 


RS： 指定读取文本时，所使用的换行符(指定以什么字符作为换行符)；默认为换行符
ORS：指定输出时所使用的行分隔符


2、awk内置变量之数据变量

NR：记录awk所处理的文本的行数，如果有多个文件，所有文件统一进行计数

[root@localhost ~]# awk '{print "第",NR,"行内容：",$0}' /etc/hosts /etc/issue
第 1 行内容： 127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
第 2 行内容： ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
第 3 行内容： CentOS release 6.6 (Final)
第 4 行内容： Kernel \r on an \m
第 5 行内容：

注意：
print在显示变量值时，不需要使用$


FNR：记录awk正在处理的文件的行数,如果有多个文件，每个文件分别进行计数

[root@localhost ~]# awk '{print "第",FNR,"行内容：",$0}' /etc/hosts /etc/issue
第 1 行内容： 127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
第 2 行内容： ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
第 1 行内容： CentOS release 6.6 (Final)
第 2 行内容： Kernel \r on an \m
第 3 行内容：


NF：记录awk正在处理的当前行被分隔成了几个字段

cat test.txt 

this is a test.

awk '{print NF}' test.txt 

4

awk '{print $NF}' test.txt 

test.

[root@localhost ~]# awk -F: '{print "Number of line: ", NF}' /etc/passwd

[root@shell ~]# awk -F. '{print "Number of Line: ", NF}' /etc/hosts
[root@shell ~]# awk 'BEGIN{FS="."}{print "Number of Line: ", NF}' /etc/hosts
Number of Line:  6
Number of Line:  3
Number of Line:  6


3、用户自定义的变量

awk允许用户自定义变量，变量名称不能以数字开头，且区分大小写

示例：

方法1：使用-v选项

[root@localhost ~]# head -n 3 /etc/passwd | awk -v test="hello" -F: '{print test, $1}'
hello root
hello bin
hello daemon



方法2：在BEGIN{}模式中定义变量

[root@localhost ~]# head -n 3 /etc/passwd | awk -F: 'BEGIN{test="hello"}{print test, $1}'
hello root
hello bin
hello daemon
[root@localhost ~]# 


​	
​	
​	


二、printf，格式化输出内容

格式：
	printf 格式,item1,item2,...

要点：
1、printf输出时要指定格式format
2、format用于指定后面的每个item输出的格式
3、printf语句不会自动打印换行符\n 

format格式：

%c：显示单个字符
%d,%i：十进制整数
%e,%E：科学计数法显示数值
%f：显示浮点数
%g,%G：以科学计数法的格式或浮点数的格式显示数值
%s：显示字符串
%u：无符号整数
%%：显示%自身

修饰符：
N：显示宽度，N为数字
-：左对齐，默认为右对齐
+：显示数值符号	


示例： 

[root@localhost ~]# head -n 3 /etc/passwd | awk -F: '{printf "%-10s%-8s%-20s\n", $1,$6,$7}'
root      /root   /bin/bash           
bin       /bin    /sbin/nologin       
daemon    /sbin   /sbin/nologin   


[root@shell ~]# df -hT | sed '1d' | awk '{printf "%-15s%-10s\n", $1,$7}'
/dev/vda2      /         
tmpfs          /dev/shm  
/dev/vda1      /boot     
[root@shell ~]# 

[root@shell ~]# awk -F: '{printf "%-15s%-10d%-25s%-15s\n",$1,$3,$6,$7}' /etc/passwd
root           0         /root                    /bin/bash      
bin            1         /bin                     /sbin/nologin  
daemon         2         /sbin                    /sbin/nologin  
adm            3         /var/adm                 /sbin/nologin  
lp             4         /var/spool/lpd           /sbin/nologin  



[root@node2 ~]# df -hT | sed '1d' | awk '{printf "%-10s%-10s%-12s%-10s%-15s%-6s%-8s%-4s\n","DiskName:", $1, "MountPoint:", $7, "TotalSize:", $3, "Usage:", $6}'
DiskName: /dev/sda3 MountPoint: /         TotalSize:     77G   Usage:  6%  
DiskName: tmpfs     MountPoint: /dev/shm  TotalSize:     491M  Usage:  0%  
DiskName: /dev/sda1 MountPoint: /boot     TotalSize:     190M  Usage:  15% 





awk [option] 'PATTERN{action}' file1 file2 ...

	action:
		print 
		printf 
		
	options: 
		-F 
		-v 


PATTERN表示方法：

1、正则表达式，格式为/regex/

以冒号为分隔符，显示/etc/passwd以r开头的行的第1段

awk -F: '/^r/{print $1}' /etc/passwd

root
rpc
rtkit
rpcuser

[root@localhost ~]# awk '/^(r|s)/{print $0}' /etc/passwd

[root@localhost ~]# awk '/^[rs]/{print $0}' /etc/passwd

[root@localhost ~]# ls -l /tmp/ | awk '/^d/{print $9}'

[root@localhost ~]# netstat -antp | awk '/^tcp/{print $0}'
[root@localhost ~]# netstat -antp | sed '1,2d'



2、表达式，由下述操作符组成的表达式

awk的操作符

1、算术操作符

-x	负值
+x	转换为数值，正值
x^y,x**y  次方	2^3    2**3      
x*y
x/y
x+y
x-y
x%y

2、字符串操作符

+：实现字符串连接		"ab"+"cd"    abcd			"ab"+"12"	ab12	

3、赋值操作符
=

+=   a+=b  			a=a+b   			a+=2	a=a+2

*=
/=
%=
^=		x ^= y     x^y       
**=

++		x++				x=x+1


4、比较操作符

x < y
x <= y
x > y
x >= y
x == y
x != y
x ~ y：x为字符串,y为模式，如果x可以被模式匹配则为真，否则为假				"abc" ~ ^a 
x !~ y


5、逻辑关系符
&&
||

显示uid大于等于500的用户名及uid 

awk -F: '$3>=500{print $1,$3}' /etc/passwd

nfsnobody 65534
wjc 500

显示默认shell为/bin/bash的用户名及shell名称

awk -F: '$7=="/bin/bash"{print $1,$7}' /etc/passwd

或

awk -F: '$7 ~ "bash$"{print $1,$7}' /etc/passwd

root /bin/bash
mysql /bin/bash
wjc /bin/bash

[root@node2 ~]# netstat -antp | awk '$6=="ESTABLISHED"{print $0}'
tcp        0     52 192.168.87.102:22           192.168.87.1:49541          ESTABLISHED 1940/sshd           
tcp        0      0 192.168.87.102:22           192.168.87.1:49650          ESTABLISHED 2135/sshd           
[root@node2 ~]# 


[root@localhost ~]# df -hT | awk '+$6>10{print $1,$NF}'
/dev/sda1 /boot
/dev/sr0 /mnt



3、指定范围，格式为pattern1,pattern2	

以冒号为分隔符，显示uid=0到最后一个字段为nologin结尾中间所有行的用户名，uid及shell

awk -F: '$3==0,$7 ~ "nologin$"{print $1,$3,$7}' /etc/passwd

root 0 /bin/bash
bin 1 /sbin/nologin

格式化输出样例

awk -F: '$3==0,$7 ~ "nologin$"{printf "%-10s%-10s%-20s\n",$1,$3,$7}' /etc/passwd

root      0         /bin/bash           
bin       1         /sbin/nologin   

4、BEGIN/END，特殊模式
	BEGIN表示awk进行处理前执行一次操作
	END表示awk处理完最后一行结束前执行一次操作

使用BEGIN打印表头

awk -F: 'BEGIN{printf "%-10s%-10s%-20s\n","Username","Uid","Shell"}$3==0,$7 ~ "nologin$"{printf "%-10s%-10s%-10s\n",$1,$3,$7}' /etc/passwd

Username  Uid       Shell               
root      0         /bin/bash 
bin       1         /sbin/nologin

使用END打印表尾

awk -F: 'BEGIN{printf "%-10s%-10s%-20s\n","Username","Uid","Shell"}$3==0,$7 ~ "nologin$"{printf "%-10s%-10s%-10s\n",$1,$3,$7}END{print "END OF File..."}' /etc/passwd

Username  Uid       Shell               
root      0         /bin/bash 
bin       1         /sbin/nologin
END OF File...











awk [option] 'PATTERN{action}' file1 file2 ... 


逻辑控制语句

1、if...else 

格式：

if(条件) {语句; 语句} else {语句1; 语句2}

如果statement只有一条语句，{}可以不写

[root@localhost ~]# awk -F: '{if($3==0){print $1,"is administrator."}}' /etc/passwd

以冒号为分隔符，判断第1个字段，如果为root，则显示用户名 Admin,否则显示用户名 Common User

awk -F: '{if ($1=="root") print $1,"Admin";else print $1,"Common User"}' /etc/passwd

root Admin
bin Common User
daemon Common User
adm Common User
lp Common User
sync Common User
shutdown Common User
格式化输出 

awk -F: '{if ($1=="root") printf "%-15s: %-15s\n",$1,"Admin";else printf "%-15s: %-15s\n",$1,"Common User"}' /etc/passwd

root           : Admin          
bin            : Common User    
daemon         : Common User    
adm            : Common User    
lp             : Common User    
sync           : Common User    
shutdown       : Common User    
halt           : Common User    

统计系统用户个数
[root@localhost ~]# awk -F: '{if($3>0 && $3<500){count++;print $1}}END{print count}' /etc/passwd

[root@shell ~]# awk -v count=0 -F: '{if($3<=500){print $1;count++}}END{print "系统用户数量：",count}' /etc/passwd
[root@shell ~]# awk -F: 'BEGIN{count=0}{if($3<=500){print $1;count++}}END{print "系统用户数量：",count}' /etc/passwd


统计LISTEN状态的TCP连接数量 

[root@test_server ~]# netstat -antp | awk -v number=0 '/^tcp/{if($6=="LISTEN") {number++}}END{print number}' 
4
[root@test_server ~]# netstat -antp | awk '/^tcp/{if($6=="LISTEN") {number++}}END{print number}' 
4


统计uid大于50的用户个数

awk -F: -v sum=0 '{if ($3>50) sum++}END{print "The number of user: ",sum}' /etc/passwd

The number of user:  16



取出磁盘使用率大于10%的

df -h | sed '1d' | awk '{if (+$5>10) {print $0}}'

/dev/sda1             194M   26M  158M  15% /boot
/dev/sr0              2.9G  2.9G     0 100% /mnt

[root@localhost ~]# awk -F: '{if($3==0){print $1}else{print $7}}' /etc/passwd

[root@localhost ~]# awk -F: '{if($3==0){count++}else{i++}}END{print "管理用户个数：",count,"普通用户个数：",i}' /etc/passwd




[root@localhost ~]# awk -F: '/bash$/ || /nologin$/{if($7=="/bin/bash"){i++} else{j++}}END{print "bash用户数量：",i, "nologin用户数量：",j}' /etc/passwd
bash用户数量： 38 nologin用户数量： 28




2、while

格式：

while(条件) {语句1; 语句2;.........}

head -n 3 /etc/passwd | awk -F: '{i=1; while(i<=3) {print $i; i++}}'


以冒号为分隔符，判断每一行的每一个字段的长度如果大于4，则显示之

awk -F: '{i=1;while (i<=7) {if (length($i)>=4) {print $i};i++}}' /etc/passwd


统计test.txt文件中长度大小5的单词

[root@localhost ~]# awk '{i=1; while(i<=NF) {if(length($i)>5) {print $i};i++}}' test.txt 

[root@shell ~]# sed 's/[[:punct:]]//g' 1.txt|awk -v count=0 '{i=1; while(i<=NF) {if(length($i)>6) {print $i; count++}; i++}}END{print "数量：" count}'


3、for			遍历数组

格式：

for(变量定义; 循环的条件; 改变循环条件的语句) {语句；语句；....}

	for(i=1;i<=4;i++) { ... }


以冒号为分隔符，显示/etc/passwd每一行的前3个字段

awk -F: '{for (i=1;i<=3;i++) {print $i}}' /etc/passwd

root
x
0
bin
x
1

4、break continue

	用于中断循环

awk中使用数组

数组基本操作：

1、定义数组

[root@test_server ~]# awk 'BEGIN{test[1]=10; test[2]=20; test[3]=30}'
[root@test_server ~]# awk 'BEGIN{test02["martin"]=1; test02["jerry"]=2; test02["mike"]=3}'


2、显示数组的数据 

[root@test_server ~]# awk 'BEGIN{ip[1]="1.1.1.1"; ip[2]="1.1.1.2"; ip[3]="1.1.1.3"; print ip[2]}'
1.1.1.2


显示数据中的所有的数据 ------  for循环 

[root@test_server ~]# awk 'BEGIN{ip[1]="1.1.1.1"; ip[2]="1.1.1.2"; ip[3]="1.1.1.3"; for(i=1; i<=3; i++) {print ip[i]}}'

[root@test_server ~]# awk 'BEGIN{ip[1]="1.1.1.1"; ip[2]="1.1.1.2"; ip[3]="1.1.1.3"; for(i=1; i<=length(ip); i++) {print ip[i]}}'

1.1.1.1
1.1.1.2
1.1.1.3

[root@test_server ~]# awk 'BEGIN{test["martin"]=10; test["jerry"]=20; test["tome"]=90; for(i in test) {print i} }'  
tome
jerry
martin
[root@test_server ~]# awk 'BEGIN{test["martin"]=10; test["jerry"]=20; test["tome"]=90; for(i in test) {print test[i]} }'  
90
20
10









数组 
array[index-expression]

数组下标从1开始，也可以使用字符串作为数组下标

index-expression可以使用任意字符串。
需要注意的是，如果某数组元素事先不存在，那么在引用其时，awk会自动创建此元素并初始化为0；因此，要判断某数组中是否存在某元素，需要使用index in array的方式

要遍历数组中每一个元素，需要使用如下的特殊结构：

for(变量 in 数组名称){print 数组名称[变量]}

其中，var是数组下标

统计每个shell的使用个数

awk -F: '{shell[$NF]++}END{for(A in shell) {print A,shell[A]}}' /etc/passwd

/bin/sync 1
/bin/bash 3
/sbin/nologin 31
/sbin/halt 1
/sbin/shutdown 1

[root@localhost ~]# awk -F: '{shells[$7]++}END{for(sh in shells){print sh,shells[sh]}}' /etc/passwd | sort -k2

{shell[$NF]}:以每行的最后的shell名称为下标，为数组元素shell[$NF]赋初始值1，如果遇到相同的shell，则该值递增1
A：代表数组的下标
print A：显示数组下标
print shell[A]：显示此下标对应的数组值，也就是个数


统计每个状态的tcp连接的个数

netstat -ant | sed '1,2d' | awk '{array[$NF]++}END{for(A in array) {print A,array[A]}}'

LISTEN 13
ESTABLISHED 1

或者

netstat -ant | awk '/^tcp/{state[$NF]++}END{for(A in state) {print A,state[A]}}' 

LISTEN 13
ESTABLISHED 1


统计某一天的访问量　

[root@localhost ~]# grep "04/Nov/2016" /var/log/httpd/access_log | wc -l
47721

统计/var/log/httpd/access_log每个IP的访问次数（UV）

[root@localhost ~]# grep "08/Mar/2017" /var/log/httpd/access_log | awk '{count[$1]++}END{for(ip in count) {print ip,count[ip]}}' | sort -n -k2 -r 

统计PV 

[root@localhost ~]# grep "08/Mar/2017" /var/log/httpd/access_log | awk '{count[$7]++}END{for(ip in count) {print ip,count[ip]}}' | sort -n -k2 -r



格式化输出：

awk 'BEGIN{printf "%-20s%-20s\n","IP Address","Access Count"}{count[$1]++}END{for(ip in count) {printf "%-20s%-20s\n",ip,count[ip]}}' /var/log/httpd/access_log 

IP Address          Access Count        
10.1.1.100          53     








awk的内置函数

int()：返回整数
[root@localhost ~]# awk 'BEGIN{print int(12.9)}'
12

index()：返回字符所在字符串中的位置
[root@localhost ~]# awk 'BEGIN{print index("abcd","b")}'
2

split(string,array [,filedsep [,seps]])
将string表示的字符串以fileldsep为分隔符进行分割，并将分隔后的结果保存至数据array中；数组下标从１开始

[root@shell ~]# awk 'BEGIN{str="This is a test"; split(str,testarray,"is");print testarray[1]}' 
Th


length(string)
返回string字符串的字符个数


[root@localhost ~]# awk 'BEGIN{str="hello world"; print length(str)}'
11

[root@localhost ~]# awk -F: '{if(length($1)==4){count++;print $1}}END{print "用户个数：",count}' /etc/passwd


substr(string,start [,length])
取string字符串中的子串，从start开始，取length个；start从1开始计数

[root@localhost ~]# awk 'BEGIN{str="this is test"; print substr(str,2,5)}'
his i

match函数，

match(字符串，子串)

检测str中是否含有"is" 如果有，则返回"is"第一次出现的位置，如果没有则返回0
[root@localhost ~]# awk 'BEGIN{str="This is a test"; print match(str,"is")}'
3

tolower(string)
将string中的所有字母转为小写

[root@localhost ~]# awk 'BEGIN{print tolower("Kello")}'
kello

toupper(string)
将string中所有字母转换为大写

[root@localhost ~]# awk 'BEGIN{print toupper("hello")}'
HELLO









