# docker 安装postgreSql

## 拉取镜像

 docker pull postgres

## 创建本地存储文件夹

 e: =>md postgres=>cd postgres=>md data

## 创建容器

docker run -itd --name postgres --restart always -e  POSTGRES_PASSWORD='123' -e ALLOW_IP_RANGE=0.0.0.0/0 -v /home/postgres/data:/e/postgres/data -p 5432:5432  postgres 

## 安装pgadmin

`docker run --name pgadmin -p 5433:80  -itd -e PGADMIN_DEFAULT_EMAIL=943885179@qq.com -e PGADMIN_DEFAULT_PASSWORD=123abc dpage/pgadmin4`
