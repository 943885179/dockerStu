# docker 安装postgresql

## 下载镜像

    docker pull postgres

## 创建容器

    docker run -it -d  --name my_postgres -e POSTGRES_PASSWORD=123 -v /var/lib/postgresql/data:/d/postgres/data -p 5432:5432 postgres

## 进入postgres

    docker exec -it my_postgres bash

## 转换到postgres用户

    psql -U postgres -W 密码 // 或者使用 su postgres

## 常用命令

    \? 查看帮助

    \l 查看数据库列表

    \c db 切换数据库

    \d 查看表

    \dt 查看表结构

    \di 查看表索引

    \q 退出

## docker进入交互式后命令和linux操作一致（exit:退出，cd:进入，ls：当前文件目录）

## 容器移除 docker stop 容器id/容器名称 (停止容器运行)=> docker rm 容器id/容器名称

## 进阶，使用上面的可以创建postgresql但是外部好像访问不到，所以需要做进一步优化

### docker cp 容器id:/var/lib/postgresql/data e:/postgres/data 将容器的data拷贝到本地

### 删除掉老的容器 docker stop 容器id=>docker rm 容器id

### 重新创建容器

    docker run -it -d  --name my_postgres -e POSTGRES_PASSWORD=123 -v /var/lib/postgresql/data:/d/postgres/data -p 5432:5432 postgres

## 创建pgadmin

    docker pull dpage/pgadmin4

    docker run -it -d --name pgadmin -p 5433:80 -e PGADMIN_DEFAULT_EMAIL=943885179@qq.com PGADMIN_DEFAULT_PASSWORD=123456 dpage/pgadmin4
You need to specify PGADMIN_DEFAULT_EMAIL and PGADMIN_DEFAULT_PASSWORD environment variables
## 访问[pgadmin](http://127.0.0.1:5433)

## 新建服务器，此时发现无法连接上postgresql，需要如下操作

    获取postgresql的ip ： docker exec -it postgres容器id bash >cat /etc/hosts 这里存储了当前的ip，使用该ip可以在pgadmin上登录了
