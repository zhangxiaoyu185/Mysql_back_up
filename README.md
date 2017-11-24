# 一、配置MySQL主服务器（192.168.21.100）

> 进入MySQL控制台

- mysql  -uroot  -p   

> 刷新系统授权表

- flush privileges;   

>授权用户xiaoyu只能从192.168.21.101访问主服务器192.168.21.100上面的数据库，并且只具有数据库备份的权限

- grant replication slave  on *.* to 'xiaoyu'@'192.168.21.101' identified by '123456' with grant option; 

# 二、主服务器192.168.21.100中的数据库导入到从服务器192.168.21.101中

>在导出之前先执行数据库只读锁定命令，防止导出数据库的时候有数据写入

- flush tables with read lock; 

>操作完成后解除锁定

- unlock tables;   

# 三、配置MySQL主服务器的my.cnf文件

>编辑配置文件

- vi /etc/my.cnf   

```
在[mysqld]中修改或添加
#设置服务器id，为1表示主服务器
server-id=1   
#启动MySQ二进制日志系统
log_bin=mysql-bin  
#需要同步的数据库名，如果有多个数据库，写多行，每个数据库一行
binlog-do-db=test
#不同步mysql系统数据库
binlog-ignore-db=mysql 
```

>保存退出

- :wq!    

>重启MySQL

- service mysqld  restart  

>进入mysql控制台

- mysql -u root -p   

>查看主服务器

- show master status; 

>出现以下类似信息
```
+-----------------+-----------+--------------+------------------+
| File            | Position  | Binlog_Do_DB | Binlog_Ignore_DB |
+-----------------+-----------+--------------+------------------+
| mysql-bin.000011|    1001   | test         | mysql            |
+-----------------+-----------+--------------+------------------+
1 row in set (0.00 sec)
```

>注意：这里记住File的值：mysql-bin.000011和Position的值：1001

# 四、配置MySQL从服务器的my.cnf文件

>编辑配置文件

- vi /etc/my.cnf  

>在[mysqld]中修改或添加
```
#配置文件中已经有一行server-id=1，修改其值为2，表示为从数据库
server-id=2 
#启动MySQ二进制日志系统  
log-bin=mysql-bin
#需要同步的数据库名，如果有多个数据库，写多行，每个数据库一行
replicate-do-db=test
#不同步mysql系统数据库
replicate-ignore-db=mysql 
```

>保存退出

- :wq!    

>重启MySQL

- service mysqld restart   

>注意：MySQL 5.1.7版本之后，已经不支持把master配置属性写入my.cnf配置文件中了，只需要把同步的数据库和要忽略的数据库写入即可

>进入MySQL控制台

- mysql  -u root -p  

>停止slave同步进程

- slave stop;   

>执行同步语句

- change master to master_host='192.168.21.100',master_user='xiaoyu',master_password='123456',master_log_file='mysql-bin.000011' ,master_log_pos=1001;   

>开启slave同步进程

- slave start;    

# 五、查看slave同步信息

- SHOW SLAVE STATUS\G  

>找到Slave_IO_Running: Yes和Slave_SQL_Running: Yes，即说明配置成功！
