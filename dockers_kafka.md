# docker kafka 配置

## docker-compose.yml

```yml
  version: '2'
  services:
    zookeeper:
      image: wurstmeister/zookeeper
      ports:
        - "2181:2181"
    kafka:
      image: wurstmeister/kafka:2.11-0.11.0.3
      ports:
        - "9092:9092"
      environment:
        KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://:9092
        KAFKA_LISTENERS: PLAINTEXT://:9092
        KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      volumes:
        - /var/run/docker.sock:/var/run/docker.sock
```

## docke-compose up -d

## 说明(上面安装后可以直接到第五步)

1. kafka需要zookeeper管理，所以需要先安装zookeeper
下载zookeeper镜像
$ docker pull wurstmeister/zookeeper
2. 启动镜像生成容器
$ docker run -d --name zookeeper -p 2181:2181 -v /etc/localtime:/etc/localtime wurstmeister/zookeeper
$ docker run -d --restart=always --log-driver json-file --log-opt max-size=100m --log-opt max-file=2  --name zookeeper -p 2181:2181 -v /etc/localtime:/etc/localtime wurstmeister/zookeeper
3. 下载kafka镜像
$ docker pull wurstmeister/kafka
4. 启动kafka镜像生成容器
$ docker run -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=172.16.0.13:2181/kafka -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://172.16.0.13:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -v /etc/localtime:/etc/localtime wurstmeister/kafka
$ docker run -d --restart=always --log-driver json-file --log-opt max-size=100m --log-opt max-file=2 --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=172.16.0.13:2181/kafka -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://172.16.0.13:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -v /etc/localtime:/etc/localtime wurstmeister/kafka
参数说明：
  -e KAFKA_BROKER_ID=0  在kafka集群中，每个kafka都有一个BROKER_ID来区分自己
  -e KAFKA_ZOOKEEPER_CONNECT=172.16.0.13:2181/kafka 配置zookeeper管理kafka的路径172.16.0.13:2181/kafka
  -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://172.16.0.13:9092  把kafka的地址端口注册给zookeeper，如果是远程访问要改成外网IP,类如Java程序访问出现无法连接。
  -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 配置kafka的监听端口
  -v /etc/localtime:/etc/localtime 容器时间同步虚拟机的时间
5. 验证kafka是否可以使用
5.1 进入容器
$ docker exec -it kafka bash

5.2 进入 /opt/kafka_2.12-2.3.0/bin/ 目录下
$ cd /opt/kafka_2.12-2.3.0/bin/

5.3、运行kafka生产者发送消息
$ ./kafka-console-producer.sh --broker-list localhost:9092 --topic sun

发送消息
> {"datas":[{"channel":"","metric":"temperature","producer":"ijinus","sn":"IJA0101-00002245","time":"1543207156000","value":"80"}],"ver":"1.0"}e
5.4、运行kafka消费者接收消息
$ ./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic sun --from-beginning
5.5  查看分组
$ ./kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list
5.6  查看分组消费情况
GROUP     TOPIC     PID       OFFSET             LOGSIZE   LAG
消费者组  话题id    分区id    当前已消费的条数   总条数    未消费的条数
$ ./kafka-consumer-groups.sh --describe --bootstrap-server localhost:9092 --group console-consumer-71508

## 检查zookeeper

1. docker exec -it kafka bash
2. cd /opt/kafka_2.11-0.11.0.3/bin/
3. ./zookeeper-shell.sh localhost:2181 <<< "get /brokers/ids/0" # 发现使用localhost无法连接，所以我就去zookeeper里看了下它的ip :cat /etc/hosts,用这个ip是可以的

## 扩展broker `docker-compose scale kafka=4`

## 单独安装

1. docker run -d --name zookeeper -p 2181:2181 -v /etc/localtime:/d/dockerfile/kafka/zookeeper/etc/localtime wurstmeister/zookeeper
2. docker run  -d --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=192.168.0.103:2181 -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.0.103:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t wurstmeister/kafka
3. 然后用上面的方式验证，#其中192.168.0.103是我本机的ip,cmd使用ipconfig查看自己的ip去做替换就可以了
4. 特别注意：我个人水平有限，没有去详细研究,所以使用docker-compose构建时候发现kafka无法连接zookeeper导致了宿主机无法访问kafka,所以还是单独创建吧，environment内容改成-e的内容试试吧,我就不试了

## go测试

```go
  package main

  import (
    "fmt"
    "github.com/Shopify/sarama"
  )

  // 基于sarama第三方库开发的kafka client

  func main() {
    fmt.Println("测试")
    config := sarama.NewConfig()
    config.Producer.RequiredAcks = sarama.WaitForAll          // 发送完数据需要leader和follow都确认
    config.Producer.Partitioner = sarama.NewRandomPartitioner // 新选出一个partition
    config.Producer.Return.Successes = true                   // 成功交付的消息将在success channel返回

    // 构造一个消息
    msg := sarama.ProducerMessage{}
    msg.Topic = "sun"
    msg.Value = sarama.StringEncoder("this is a test log")
    // 连接kafka
    fmt.Println("连接kafka")
    client, err := sarama.NewSyncProducer([]string{"127.0.0.1:9092"}, config)
    if err != nil {
      fmt.Println(err)
      return
    }
    defer client.Close()
    fmt.Println("发送kafka")
    // 发送消息
    pid, offset, err := client.SendMessage(&msg)
    if err != nil {
      fmt.Println(err)
      return
    }
    fmt.Println(pid, offset)
  }
```
