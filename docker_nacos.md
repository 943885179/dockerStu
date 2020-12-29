# docker nacos安装

## 文档已经很详细了[文档](https://nacos.io/zh-cn/docs/quick-start-docker.html)

1. git clone https://github.com/nacos-group/nacos-docker.git =>cd nacos-docker
2. docker-compose -f example/standalone-derby.yaml up #用什么数据库随便了

## 可能出现的问题 grafana/grafana 拉去失败，单独先docker pull grafana/grafana再创建容器

## 成功访问[控制台](http://127.0.0.1:8848/nacos/),默认账号密码都是nacos
