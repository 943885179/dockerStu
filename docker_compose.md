# docker-compose 简单说明

## 安装，windows已经自带了，linux下安装

### 从github上下载docker-compose二进制文件安装

1. 下载docker-compose二进制文件安装 :sudo curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
2. 若是github访问太慢，可以用daocloud下载 sudo curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
3. 添加可执行权限 sudo chmod +x /usr/local/bin/docker-compose
4. 测试 docker-compose --version

### 使用pid安装

1. sudo pip install docker-compose

## docker-compose文件结构和示例

```yml
version "1" # 版本号
services: # 服务
  test1: #第一个服务
    container_name: "容器名称"
    image:  "redis:alpine" # 以来images
    ports: --端口
      - "6379:6379"
      - "3000"
      - "3000-3005"
      - "8000:8000"
      - "9090-9091:8080-8081"
      - "49100:22"
      - "127.0.0.1:8001:8001"
      - "127.0.0.1:5000-5010:5000-5010"
      - "6060:6060/udp"
    volumes: # 卷挂载
      - /user/etc:/d/user/etc
    environment: ## 添加环境变量。 你可以使用数组或字典两种形式。 任何布尔值; true，false，yes，no需要用引号括起来，以确保它们不被YML解析器转换为True或False。 
      RACK_ENV: development
      SHOW: 'true'
      - SHOW=true ## 也可以使用该方式去赋值
    command: bundle exec thin -p 3000 #该命令也可以是一个类似于dockerfile的列表：
    links: #链接到另一个服务中的容器
      - db 
    external_links: # 链接到docker-compose.yml 外部的容器
     - ss
    expose: #暴露端口，但不映射到宿主机，只被连接的服务访问。
      - "3000"
      - "8000"
    extra_hosts: #添加主机名映射。类似 docker client --add-host。 会在此服务的内部容器中 /etc/hosts 创建一个具有 ip 地址和主机名的映射关系 cat /etc/hosts
      - "somehost:162.242.195.82"
      - "otherhost:50.31.209.229"
    logging:
      driver: syslog # 指定服务容器的日志记录驱动程序，默认值为json-file "json-file" "syslog" "none"
      options:
        max-size: "200k" # 单个文件大小为200k
        max-file: "10" # 最多10个文件
        syslog-address: "tcp://192.168.0.42:123"  # syslog 驱动程序下，可以使用 syslog-address 指定日志接收地址。
  test2: #第二个服务
    build: ./dir ##以来的dockerfile路径，会在该文件夹下读取Dockerfile文件
    ports:
      - target: 80
        published: 8080
        protocol: tcp
        mode: host
     #...
  test2: #第三个服务
    build: # 会着./dir文件夹下的Dockerfile_test的Dockerfile文件作为Dockerfile
      context: ./dir
      dockerfile: Dockerfile_test
      args:
        buildno: 1 ## 可读取参数，可设置多个 Dockerfile内容为
            ## ARG buildno
            ## RUN echo "Build number: $buildno"   
    ports: --端口
      - "6379"

```

## 其他的参数参照[文档](https://www.runoob.com/docker/docker-compose.html)

## 编译

1. docker-compose up -d 会执行查找命名为docker-compose.yml文件
2. 如果yml文件名称不是docker-compose.yml 则用`docker-compose -f xxx.yml up -d`
