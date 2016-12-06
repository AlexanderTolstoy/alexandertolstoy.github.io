---
title: remove 命令参数列表过长导致无法删除的扩展脚本
date: 2016-11-24 10:31:14
categories:
- OS
    tags: 
- Linux  
- remove
- bash
    ---

## 需求与设计 ##

Oracle 11g 默认开启审计功能,另外系统也会产生各种日志文件,如log,alert,trace等,文件不大,但是占用inode,也会导致磁盘使用率爆满,积压过久,就会产生单个目录下文件太多,用remove命令删除 导致文件列表太长太大无法执行的问题,又不能直接删除上级目录。有鉴于此,写个shell脚本专门解决此类问题。

文件名   rm.sh
参数列表 : d(directory)=directoryPath  t(type)=alpha,num,alpnum  n(filename)=constantInFilename  e=fileExtension
调用示例 : rm.sh d=/u01/app/oracle/rmdbs/trace t=num n=threa1_ e=trm
实际执行 : rm -rf /u01/app/oracle/rdmbs/trace/threa1_${num}*.trm
设计 	 : 如果 无法删除, 生成含正则表达式的 rm 参数 如 *${num}* 如果仍删除失败,累加正则  如 *${num}*${num}* 以此类推,最好采用递归实现.


## 脚本实现 ##

### 设计/流程/为代码 ###

```bash
#文件头 //环境 如!/bin/sh  encode 等
#文件说明 //做什么的,参数列表,返回值等
#参数处理  //接受参数,参数确值, 传惨异常处理
#递归执行删除如遇无法删除增加正则表达式
#执行完成,回报结果,0/1
```

``` mermaid
graph TB
S0((0)) --> P11[参数处理] ;
P11 -.-> P111[接收参数] ;
P111 -.-> P112[参数处理<br>可空非空约束,值域判断] ;
P112 -.-> D111{是否异常 } ;
D111 -. 目录不能为空值<br>或不能为根目录 .-> E1 ;
D111 -. 变量类型未指定 .-> E1 ;
D111 -. 符合参数规范 .-> P21;
P11 --> P21[执行本轮/本层rm <br>由0层级开始0代表变量数为0] ;
P21 --> D11{是否执行成功} ;
D11 -- 其他异常 --> E1((1)) ;
D11 -- 成功删除 -->  D21{当前层级<br>是否为0级} ;
D11 -- rm 参数列表过长 --> P41[rm 参数变量数目 +1<br>进入下一层开启删除] ;
D21 -- 是0级 --> E2((0)) ;
D21 -- 非0级 --> P31[rm 参数变量数目-1 <br>返回上一层继续删除] ;
P31 --> P21 ;
P41 --> P21 ;

```

```bash
rm /u01/app/oracle/rdbms/trace/*.trm --> 1
rm /u01/app/oracle/rdbms/trace/*1*.trm --> 0
rm /u01/app/oracle/rdbms/trace/*2*.trm --> 0
rm /u01/app/oracle/rdbms/trace/*3*.trm --> 1
rm /u01/app/oracle/rdbms/trace/*3*1*.trm --> 0
rm /u01/app/oracle/rdbms/trace/*3*2*.trm --> 0
...-->
rm /u01/app/oracle/rdbms/trace/*3*9*.trm --> 0
rm /u01/app/oracle/rdbms/trace/*4*.trm --> 1
rm /u01/app/oracle/rdbms/trace/*4*1*.trm --> 1
rm /u01/app/oracle/rdbms/trace/*4*1*1*.trm --> 0
rm /u01/app/oracle/rdbms/trace/*4*1*2*.trm --> 0
...-->
rm /u01/app/oracle/rdbms/trace/*4*1*9*.trm --> 0
rm /u01/app/oracle/rdbms/trace/*4*2*.trm --> 0
rm /u01/app/oracle/rdbms/trace/*4*3*.trm --> 0
...-->
rm /u01/app/oracle/rdbms/trace/*4*9*.trm --> 0
rm /u01/app/oracle/rdbms/trace/*5*.trm --> 0
...-->
rm /u01/app/oracle/rdbms/trace/*9*.trm --> 0
rm /u01/app/oracle/rdbms/trace/*.trm --> 0
```

