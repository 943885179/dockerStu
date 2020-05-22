# 下载mysql镜像`docker pull mysql`
# 查询是否成功下载 `docker images`
# 启动容器 `docker run --name dockermysql -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -d mysql`  这里配置了mysql密码为123456
   - -d 后台运行 成功启动容器后输出容器的完整ID.
   - –name：给新创建的容器命名，此处命名为ly-mysql
   - -e：配置信息
   - -p 端口映射 将主机的端口映射到容器的一个端口 主机端口:容器内部的端口
   - -it         #  是-i  和 -t的简写， 表示以交互式的方式运行容器，加上-d表示后台运行，这里为了截图输出启动日志我用了-it，也可以用-d，再用"docker logs 容器名"命令输出日志
   - --rm        #当容器被停止时自动删除容器
   - -p 8888:80  #80是为容器中的tomcat设置的端口， 这里表示将80映射到宿主机8888端口， 如果只写-p 80  容器会随机取值32768~61000中较大的端口号来映射到80端口上
   - v     # 将tomcat中的usr/local/tomcat/webapps目录映射到宿主机当前目录的webapps目录，后面更新jar包直接扔到被映射的宿主机目录中即可
# 查看运行中的容器 `docker ps`
# 此时一般可以连接了，特殊情况有报错
  + 连接mysql 8提示2059 - authentication plugin 'caching_sha2_password... MySQL 版本太新造成
    - 进行mysql编辑界面`docker exec -it dockermysql bash`
    - 登录mysql `mysql -uroot -p`
    - 修改密码 `ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'newpassword';`这个newpassword是自己设置的新密码
    - 查看数据库 `show databases;`
    - 再用navicat登录即可成功
# 备份数据库
1.查看当前启动的mysql运行容器
docker ps   
2.使用以下命令备份导出数据库中的所有表结构和数据
docker exec -it  mysql mysqldump -uroot -p123456 paas_portal > /cloud/sql/paas_portal.sql  
3.只导数据不导结构
    mysqldump　-t　数据库名　-uroot　-p　>　xxx.sql　
docker exec -it mysql mysqldump -t -uroot -p123456 paas_portal >/cloud/sql/paas_portal_dml.sql  
4.只导结构不导数据
mysqldump　--opt　-d　数据库名　-u　root　-p　>　xxx.sql　
docker exec -it mysql mysqldump  --opt -d   -uroot -p123456 paas_portal >/cloud/sql/paas_portal_ddl.sql   
5.导出特定表的结构
mysqldump　-uroot　-p　-B　数据库名　--table　表名　>　xxx.sql
docker exec -it mysql mysqldump -uroot -p -B paas_portal --table user > user.sql  