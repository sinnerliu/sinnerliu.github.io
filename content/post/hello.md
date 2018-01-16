---
title: "linux安装oracle精简版客户端"
date: 2018-01-16T11:52:41+08:00
lastmod: 2018-01-16T11:52:41+08:00
draft: false
keywords: [oracle]
description: ""
tags: [数据库]
categories: [oracle12]
author: "死性不改"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: true
autoCollapseToc: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
---

linux安装oracle精简版客户端
<!--more-->

###  前提准备

安装包下载地址：[http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html)

版本要求：最好和服务端一致，可以高一点，但是不能底于服务端的版本号；最好不要高于oracle服务端的的大版本，比如，服务端是11g的，最好就下11版本的客户端，不要下载12版本的客户端；

下载包：
oracle-instantclient11.2-basic-11.2.0.1.0-1.x86_64.zip
oracle-instantclient11.2-jdbc-11.2.0.1.0-1.x86_64.zip 
oracle-instantclient11.2-sqlplus-11.2.0.1.0-1.x86_64.zip
oracle-instantclient11.2-odbc-11.2.0.1.0-1.x86_64.zip
oracle-instantclient11.2-tools-11.2.0.1.0-1.x86_64.zip

### 解压

把上述包全部解压

```shell
unzip oracle-instantclient11.2-basic-11.2.0.1.0-1.x86_64.zip
unzip oracle-instantclient11.2-jdbc-11.2.0.1.0-1.x86_64.zip 
unzip oracle-instantclient11.2-sqlplus-11.2.0.1.0-1.x86_64.zip
unzip oracle-instantclient11.2-odbc-11.2.0.1.0-1.x86_64.zip
unzip oracle-instantclient11.2-tools-11.2.0.1.0-1.x86_64.zip
```

解压后得到目录：instantclient_11_2

赋执行权限

```shell
chmod -R 7755 instantclient_11_2
```

### 配置环境变量

````shell
vi ~/.bash_profile
export ORACLE_HOME=/home/oracle/instantclient_11_2
export ORACLE_IC_HOME=$ORACLE_HOME
export TNS_ADMIN=$ORACLE_HOME
export NLS_LANG=AMERICAN_AMERICA.AL32UTF8
export LD_LIBRARY_PATH=$ORACLE_HOME:$LD_LIBRARY_PATH
export CLASSPATH=$ORACLE_HOME/ojdbc6.jar:./
export PATH=$PATH:$ORACLE_HOME
export ORACLE_SID=orcl
````

使环境变量生效

```shell
source ~/.bash_profile
```

### 配置tnsname.ora

```shell
cd $ORACLE_HOME
vi tnsname.ora
orcl =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 192.168.132.2 )(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVICE_NAME = orcl)
    )
  )
```

### 测试sqlplus

```shell
sqlplus oracle/oracle@orcl
SQL*Plus: Release 11.2.0.1.0 - Production on Fri Aug 5 12:32:54 2005

Copyright (c) 1982, 2005, Oracle. All rights reserved.

SQL>
```

连接成功！

如果sqlplus报错：ORA-12533 或者 ORA-12533: TNS:illegal ADDRESS parameters

解决办法：

* 先检查tnsname.ora有没有配置错误；
* 如果tnsname.ora确定正确，则需要把tnsname.ora文件里面的所有空格和换行符去掉，里面只剩下一行，如下：
```shell
orcl=(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=192.168.132.2)(PORT=1521)))(CONNECT_DATA=(SERVICE_NAME=orcl)))
```
然后在进行测试，应该会成功

创建目录(后面的tnsping、exp、imp、sqlldr会用到)：

```shell
mkdir -p $ORACLE_HOME/network/mesg/
mkdir -p $ORACLE_HOME/rdbms/mesg
```

### tnsping命令

由于精简版客户端默认没有tnsping的，需要我们去服务端copy；

```shell
scp $ORACLE_HOME/bin/tnsping oracle@ip_client:$ORACLE_HOME/tnsping
scp $ORACLE_HOME/network/mesg/tnsus.msb oracle@ip_client:$ORACLE_HOME/network/mesg/tnsus.msb
```

然后再测试tnsping

```shell
#tnsping orcl
TNS Ping Utility for Linux: Version 11.2.0.1.0 - Production on 10-MAR-2014 09:28:39
Copyright (c) 1997, 2006, Oracle.  All rights reserved.
Used parameter files:
Used HOSTNAME adapter to resolve the alias
Attempting to contact (DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=192.168.132.2)(PORT=1521)))(CONNECT_DATA=(SERVICE_NAME=orcl)))
```

### exp和imp命令

由于精简版客户端默认没有exp和imp命令，需要我们去服务端copy；

```shell
scp $ORACLE_HOME/bin/exp oracle@ip_client:$ORACLE_HOME/exp
scp $ORACLE_HOME/bin/imp oracle@ip_client:$ORACLE_HOME/imp
scp $ORACLE_HOME/rdbms/mesg/ulus.msb oracle@ip_client:$ORACLE_HOME/rdbms/mesg/ulus.msb
scp $ORACLE_HOME/rdbms/mesg/expus.msb oracle@ip_client:$ORACLE_HOME/rdbms/mesg/expus.msb
scp $ORACLE_HOME/rdbms/mesg/impus.msb oracle@ip_client:$ORACLE_HOME/rdbms/mesg/impus.msb
```

### sqlldr命令

由于精简版客户端默认没有sqlldr命令，需要我们去服务端copy；

```shell
scp $ORACLE_HOME/bin/sqlldr oracle@ip_client:$ORACLE_HOME/sqlldr
scp $ORACLE_HOME/network/mesg/* oracle@ip_client:$ORACLE_HOME/network/mesg/
scp $ORACLE_HOME/rdbms/mesg/* oracle@ip_client:$ORACLE_HOME/rdbms/mesg
scp $ORACLE_HOME/lib/* oracle@ip_client:$ORACLE_HOME/
```

你会发现network/mesg/和rdbms/mesg/下面copy的东西把上面tnsping、exp、imp中copy的文件都包含了；这个其实是免得有些库依赖或者.msb没有copy完全，索性就全部copy过来了，免得后面再出问题；偷了个懒！

为了放置copy过来的文件没有执行权限，我们在客户端服务器上再进行一次赋权

```shell
chmod -R 7755 $ORACLE_HOME
```

然后你再去测试sqlldr命令就会发现已经能用了；

```shell
$ sqlldr

SQL*Loader: Release 11.2.0.1.0 - Production on Sat Jan 6 14:17:09 2018

Copyright (c) 1982, 2009, Oracle and/or its affiliates.  All rights reserved.


Usage: SQLLDR keyword=value [,keyword=value,...]

Valid Keywords:

    userid -- ORACLE username/password           
   control -- control file name                  
       log -- log file name                      
       bad -- bad file name                      
      data -- data file name                     
   discard -- discard file name                  
discardmax -- number of discards to allow          (Default all)
      skip -- number of logical records to skip    (Default 0)
      load -- number of logical records to load    (Default all)
    errors -- number of errors to allow            (Default 50)
      rows -- number of rows in conventional path bind array or be
```

### 最后

最后还有一件重要的事情，去喝杯茶，抽支烟，犒劳下自己！