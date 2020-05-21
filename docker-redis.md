#  拉取镜像 
`docker pull redis`
# 部署容器
`docker run -itd --name redis-test -p 6379:6379 redis`
# 下载Redis Desktop Manager https://pan.baidu.com/s/1Jvr9MbgFn4UJh4M1AMo3gA 提取码：3i9b
# 傻瓜式安装，host输入127.0.0.1，测试连接
-- 此时可以使用了
# 准备redis的一些配置文件

首先在/root/redis/data 创建好文件夹用于存放redis数据，这个文件夹位置也可以自己选。

然后在/root/redis/ 创建好redis.conf文件。用户redis的配置。redis.conf可以从redis官网下载 然后启动的时候导入redis的配置文件，就可以按照配置来启动了。
mkdir /root/redis
mkdir /root/redis/data
# 下载reids配置文件
`wget https://raw.githubusercontent.com/antirez/redis/5.0/redis.conf -O redis.conf`
# redis配置文件修改

    redis配置文件详解

    我的配置是直接注释掉bind

    protected-mode yes

    其他配置未改动

启动过程中遇到了一个问题所以修改了一下 echo 511 > /proc/sys/net/core/somaxconn

我用的是vm  为了链接虚拟机中的docker 在网上查了一下

    在Windows宿主机中连接虚拟机中的Docker容器

具体可以查看

    https://www.cnblogs.com/linux-wangkun/p/5840441.html
# docker 启动redis

docker run --name myredis -p 6379:6379 -d redis:latest redis-server

docker run --name myredis --restart=always -i -t -p 192.168.31.131:6379:6379 -d redis:latest redis-server

docker run --privileged=true -p 192.168.31.131:6379:6379 -v /root/redis/data:/data -v /root/redis/conf/redis.conf:/etc/redis/redis.conf  --name myredis --restart=always -d redis redis-server /etc/redis/redis.conf

    -p 6379:6379:把容器内的6379端口映射到宿主机6379端口

    -v /root/redis/redis.conf:/root/redis/redis.conf：把宿主机配置好的redis.conf放到容器内的这个位置中

    -v /root/redis/data:/data：把redis持久化的数据在宿主机内显示，做数据备份

    redis-server /etc/redis/redis.conf：这个是关键配置，让redis不是无配置启动，而是按照这个redis.conf的配置启动

    –appendonly yes：redis启动后数据持久化
启动过程中发现 执行后docker ps 查不到redis 解决方法

    --privileged=true 增加权限

如果出现重复name 使用

  docker ps -a  查看

  docker rm <containerid/names> 移除镜像
# 进入客户端 
`docker exec -it  dockerredis redis-cli -h localhost -p 6379`
>进入客户端后愉快的玩耍把
`set key1 value1`
`set key1`
`del key1`
# 可以配置可视化工具

RedisDesktopManager

官网下载：https://redisdesktop.com/download

github地址：https://github.com/uglide/RedisDesktopManager/releases

百度网盘：https://pan.baidu.com/s/1TzLn8DqbuKf57xa3pa2FYw

傻瓜式安装

点击connect to redis server

填写完成host及端口号可以点击test connection 测试一下是否连接成功。