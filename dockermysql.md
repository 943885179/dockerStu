# docker mysql

## 下载imgage

`docker pull mysql`

## 创建容器

>创建容器

`docker run -d --name [mysqlName] -p port:3306 -e MYSQL_ROOT_PASSWORD=123 mysql`

>查看容器是否启动

`docker ps`

>进入容器登录并且修改密码，否则navicat无法连接

```cmd
#进入容器
docker exec -it [mysqlName] bash
#登录mysql
mysql -uroot -p
#修改密码
alter user 'root'@'%' identified with mysql_native_password by '[newPassword]'
```

## 备份数据库

1.查看当前启动的mysql运行容器 docker ps
2.使用以下命令备份导出数据库中的所有表结构和数据 docker exec -it mysql mysqldump -uroot -p123456 paas_portal > /cloud/sql/paas_portal.sql
3.只导数据不导结构 mysqldump　-t　数据库名　-uroot　-p　>　xxx.sql　 docker exec -it mysql mysqldump -t -uroot -p123456 paas_portal >/cloud/sql/paas_portal_dml.sql
4.只导结构不导数据 mysqldump　--opt　-d　数据库名　-u　root　-p　>　xxx.sql　 docker exec -it mysql mysqldump --opt -d -uroot -p123456 paas_portal >/cloud/sql/paas_portal_ddl.sql
5.导出特定表的结构 mysqldump　-uroot　-p　-B　数据库名　--table　表名　>　xxx.sql docker exec -it mysql mysqldump -uroot -p -B paas_portal --table user > user.sql

## 主从复制

### 根据上面创建容器的方式再创建一个slave

#### 登录主服务器

>添加vim编辑器
>>`apt-get update`然后`apt-get install vim`否则直接`apt-get install vim`会报错
>>使用 `vi`查看是否成功
>编辑my.cnf
>>`cd etc/mysql`
>>`vi my.cnf`

```ini
[mysqld]
## 同一局域网内注意要唯一
server-id=100  
## 开启二进制日志功能，可以随便取（关键）
log-bin=mysql-bin
```

>创建数据同步用户，授予用户 slave REPLICATION SLAVE权限和REPLICATION CLIENT权限，用于在主从库之间同步数据。
>>`CREATE USER 'slave'@'%' IDENTIFIED BY '123456';`
>>`GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';`
> 重启服务登录mysql然后查看`show master status;`记录下File和Position字段

#### 查看主服务器的docker容器独立ip `docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器名称|容器id`

#### 登录从服务器

>添加vim
>编辑my.cnf

```ini
[mysqld]
## 设置server_id,注意要唯一
server-id=101  
## 开启二进制日志功能，以备Slave作为其它Slave的Master时使用
log-bin=mysql-slave-bin
## relay_log配置中继日志
relay_log=edu-mysql-relay-bin
```

>重启服务登录mysql(从)执行`change master to master_host='172.17.0.2', master_user='slave', master_password='123456', master_port=3306, master_log_file='mysql-bin.000001', master_log_pos= 156, master_connect_retry=30;`
>>master_host ：Master的地址，指的是容器的独立ip,可以通过  `docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器名称|容器id`查询容器的ip
>>master_port：Master的端口号，指的是容器的端口号
>>master_user：用于数据同步的用户
>>master_password：用于同步的用户的密码
>>master_log_file：指定 Slave 从哪个日志文件开始复制数据，即上文中提到的 File 字段的值
>>master_log_pos：从哪个 Position 开始读，即上文中提到的 Position 字段的值
>>master_connect_retry：如果连接失败，重试的时间间隔，单位是秒，默认是60秒
>执行`show slave status \G;`
>>正常情况下，SlaveIORunning 和 SlaveSQLRunning 都是No，因为我们还没有开启主从复制过程
>>启动主从复制`start slave;`
>>再次执行 `show slave status \G;`SlaveIORunning 和 SlaveSQLRunning 都是Yes，说明主从复制已经开启。此时可以测试数据同步是否成功
>>如果SlaveIORunning为Connecting则说明主从复制一直处于连接状态我们可以根据 Last_IO_Error提示予以排除。
>>>网络不通----检查ip,d=端口
>>>密码不对----检查用户名密码
>>>pos不对----检查Master的 Position

#### 测试主从复制

在主服务器创建一个数据库看从服务器看是否有就可以了

## 多主多从设置

### 主一配置

#### my.cnf `vi etc/mysql/my.cnf`

```ini
#主服务器唯一ID(必须唯一)
server-id=101
#启用二进制日志
log-bin=mysql-bin
# 设置不要复制的数据库(可设置多个)
binlog-ignore-db=mysql
binlog-ignore-db=information_schema
#设置需要复制的数据库
binlog-do-db=需要复制的主数据库名字
#设置logbin格式
binlog_format=STATEMENT

# 在作为从数据库的时候，有写入操作也要更新二进制日志文件
log-slave-updates
#表示自增长字段每次递增的量，指自增字段的起始值，其默认值是1，取值范围是1 .. 65535 
auto-increment-increment=2
# 表示自增长字段从哪个数开始，指字段一次递增多少，他的取值范围是1 .. 65535 
auto-increment-offset=1
```

#### 重启服务，然后创建授权账户

>`grant replication slave on *.* to 'slave'@'% identified by 'passoword'`这个语句好像报错，分开执行吧(下面两个步骤)
>`create user 'slave'@'%' identified by 'password'`
>`grant replication slave,replication client on *.* to 'slave'@'%'`
>查看创建的用户`select * from mysql.user where user='username';`

### 主二设置

#### my.ini  `vi etc/mysql/my.cnf`

```ini
#服务器唯一ID
server-id=201

log-bin=mysql-bin
binlog-ignore-db=mysql
binlog-ignore-db=information_schema
binlog-do-db=需要复制的主数据库名字
#设置logbin格式
binlog_format=STATEMENT
log-slave-updates
auto-increment-increment=2
# 主2的起始值不能和主一的一样 这里设置成2
auto-increment-offset=2
```

#### 主二重启服务，然后创建授权账户

>`grant replication slave on *.* to 'slave'@'% identified by 'passoword'`这个语句好像报错，分开执行吧(下面两个步骤)
>`create user 'slave'@'%' identified by 'password'`
>`grant replication slave,replication client on *.* to 'slave'@'%'`
>查看创建的用户`select * from mysql.user where user='username';`

### 双主依赖互建

>主服务器(172.17.0.2)

```cmd
 change master to
 master_host='172.17.0.2',
 master_user='slave',
 master_password='123456',
 master_log_file='mysql-bin.000001',
 master_log_pos=721,
 master_connect_retry=60;
```

`change master to master_host='172.17.0.2', master_user='slave', master_password='123456', master_port=3306, master_log_file='mysql-bin.000001', master_log_pos= 721, master_connect_retry=30;`

>主2服务器(172.17.0.3)

```cmd
 change master to
 master_host='172.17.0.3',
 master_user='slave',
 master_password='123456',
 master_log_file='mysql-bin.000004',
 master_log_pos=156,
 master_connect_retry=60;
```

最后一个参数超时时间调大点，不然老失败
`change master to master_host='172.17.0.2', master_port=3306, master_log_file='mysql-bin.000004',master_log_pos= 156,master_user='slave',master_password='123456', master_connect_retry=30;`

### 从一

>my.cnf

```ini
    server-id=3308
    #启用中继日志
    relay-log=mysql-relay
```

`change master to master_host='172.17.0.2', master_port=3306, master_log_file='mysql-bin.000006',master_log_pos= 156,master_user='slave',master_password='123456', master_connect_retry=30;`

### 从二

>my.cnf

```ini
    server-id=102
    #启用中继日志
    relay-log=mysql-relay
```

`change master to master_host='172.17.0.3', master_port=3306, master_log_file='mysql-bin.000003',master_log_pos= 156,master_user='slave',master_password='123456';`

## 设置完后注意不要重启docker了，否则ip，pos等有变动，需要重新配置
