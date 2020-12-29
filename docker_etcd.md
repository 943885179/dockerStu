# docker etcd 部署

## etcd 占用端口：2379/4001,2380,最新的好像是4001

## 查看docker镜像 `docker search etcd`

## 下载etcd images `docker pull elcolio/etcd / docker pull bitnami/etcd`
  下载运行后发现我使用的是go-micro,它要v3版本以上，默认装的好像是v2，所以就注入不了了。，后面就改用了官方的镜像 `docker pull quay.io/coreos/etcd`
  运行etcd `docker run -it -d -p 2379:2379 -p 2380:2380 --name etcd1 elcolio/etcd /  docker run -it --name etcd bitnami/etcd`

## 创建容器
  
   `docker run -it -d   -p 4001:4001 -p 2379:2379 -p 2380:2380   -v /etc/ssl/certs/:/d/dockerfile/etcd/etc/ssl/certs/  --name etcd1    quay.io/coreos/etcd    --listen-client-urls http://0.0.0.0:2379   --advertise-client-urls http://0.0.0.0:2379  --listen-peer-urls http://0.0.0.0:2380  --initial-advertise-peer-urls http://0.0.0.0:2380  --initial-cluster my-etcd-1=http://0.0.0.0:2380 --initial-cluster-token tkn  --initial-cluster-state new `
  不知道为啥，这些参数执行有问题，所以换成用docker-compose执行看看

## 进入容器 `docker exec -it etcd1 sh`

  查看版本 etcd --version
  查看etcdctl版本 etcdctl -v
  添加key etcdctl set foo test
  --ttl '0'            该键值的超时时间（单位为秒），不配置（默认为 0）则永不超时
  --swap-with-value value 若该键现在的值是 value，则进行设置操作
  --swap-with-index '0'    若该键现在的索引值是指定索引，则进行设置操作
  查看key etcdctl get foo
  修改key etcdctl update foo 123
  查看列表 etcdctl ls
  删除key etcdctl rm foo
  mk 如果给定的键不存在，则创建一个新的键值

## 发现了个问题，上面的安装成功呢后不知道为啥好像参数不对吧，然后外部访问不了对应的端口，改用docker-compose 去构建

### 创建docker-compose.yml文件

```yml
version: "3.5"
services:
  etcd:
    container_name: etcd1
    # hostname: etcd
    image: quay.io/coreos/etcd # bitnami/etcd:3
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
    ports:
      - "2379:2379"
      - "2380:2380"
      - "4001:4001"
    #   - "7001:7001"
    user: root
    # volumes:
    #  - "/d/dockerfile/etcd/etc:/opt/bitnami/etcd/data"
    environment:
      - "ETCD_ADVERTISE_CLIENT_URLS=http://etcd:2379"
      - "ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379"
      - "ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380"
      - "ETCD_INITIAL_ADVERTISE_PEER_URLS=http://0.0.0.0:2380"
      - "ALLOW_NONE_AUTHENTICATION=yes"
      - "ETCD_INITIAL_CLUSTER=node1=http://0.0.0.0:2380"
      - "ETCD_NAME=node1"
      - "ETCD_DATA_DIR=/opt/bitnami/etcd/data"
      - "ETCDCTL_API=3"
    networks:
      - etcdnet

networks:
  etcdnet:
    name: etcdnet
```

## 执行docker-compose.yml文件 `docker-compose up -d` 如果不是docker-compose.yml文件。使用`docker-compose -f xxx.yml up -d`

## 使用etcdctl -v 可以查看etcdctl版本和暴露的api版本`API version: 2`

1. 添加key-value :etcdctl set foo 123
2. 在cmd查看keys `curl -L http://127.0.0.1:2379/v2/keys`
3. 者浏览器访问`http://127.0.0.1:2379/v2/keys/foo`

## 然后我要将go-micro注入到etcd,发现正常启动了，也能找到服务，但是在etcd上看不到服务，所以换了个images ,发现是api版本为v2，所以换成bitnami/etcd:3默认的是v3,或者执行命令`export ETCDCTL_API=3`，不过这样会导致一个问题就是关闭后又恢复到v2了,所以直接在environment添加了etcdctl_api=3指定即可，下面的方式也可以

```yml
version: "3.5"
services:
  etcd:
    container_name: etcd1
    # hostname: etcd
    image:  bitnami/etcd:3 # quay.io/coreos/etcd #
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
    ports:
      - "2379:2379"
      - "2380:2380"
      - "4001:4001"
    #   - "7001:7001"
    user: root
    # volumes:
    #  - "/d/dockerfile/etcd/etc:/opt/bitnami/etcd/data"
    environment:
      - "ETCD_ADVERTISE_CLIENT_URLS=http://etcd:2379"
      - "ETCD_LISTEN_CLIENT_URLS=http://0.0.0.0:2379"
      - "ETCD_LISTEN_PEER_URLS=http://0.0.0.0:2380"
      - "ETCD_INITIAL_ADVERTISE_PEER_URLS=http://0.0.0.0:2380"
      - "ALLOW_NONE_AUTHENTICATION=yes"
      - "ETCD_INITIAL_CLUSTER=node1=http://0.0.0.0:2380"
      - "ETCD_NAME=node1"
      - "ETCD_DATA_DIR=/opt/bitnami/etcd/data"
    networks:
      - etcdnet

networks:
  etcdnet:
    name: etcdnet
```

## V3版本取消了ls等，命令方式也有所不同

1. etcdctl get / --prefix
2. etcdctl version
3. etcdctl put key value
4. etcdctl get key
5. etcdctl del key
6. etcdctl -h
7. v3版本是用rpc调用的，比http更加高效

## etcd ui 界面 e3w

```yml
version: "3.5"
services:
  e3w:
    hostname: e3w
    image: soyking/e3w:latest
    deploy:
      replicas: 1
      restart_policy:
        condition: on-failure
    # ports:
    #   - "8080:8080"
    volumes:
      - "/var/docker/e3w/conf/config.ini:/app/conf/config.default.ini"
    networks:
      - e3wnet
      - etcdnet

networks:
  e3wnet:
    name: e3wnet
  etcdnet:
    external: true
```

//有些莫名其妙。为啥读取不到呢，后续再慢慢研究是不是版本的问题
