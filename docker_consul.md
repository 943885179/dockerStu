[TOC]
#查看镜像是否存在 `docker search consul`
#下载镜像 `docker pull consul`
#单个server启动
### 创建server `docker run -d  -p 8500:8500 -h node1 --name node1 consul agent -server -bootstrap-expect=1 -node=node1 -client 0.0.0.0 -ui`
### 查询server的ip `docker inprece node1`查看IPaddress
### 创建client `docker run -d -p 8600:8600 -P 8500:8500 -P 8600:53/udp --name client1 -h client1 consul -ui -node=client1 -join ipaddress `
### `consul agent -data-dir=/tmp/agent -node=clinet1 -bind=10.30.0.52 -ui -dc=dc1`  ##不使用docker运行的clinet端
### 进入`docker exec -it servernode1 /bin/sh`
### 查看服务server 'consul members'
#启动镜像2多集群） 
 ```
  //启动server
  docker run -d --name=node1 --restart=always -e 'CONCUL_LOCAL_CONFIG={"skip_leave_on_interrupt":true}' -p 8300:8300 -p 8301:8301 -p 8301:8301/udp -p 8302:8302/udp -p 8302:8302 -p 8400:8400 -p 8500:8500 -p 8600:8600 -h node1 consul agent -server -bind=172.17.0.2 -bootstrap-expect=3 -node=node1 -data-dir=/tem/data-dir -client 0.0.0.0 -ui
  docker run -d --name=node2 --restart=always \
             -e 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true}' \
             -p 9300:8300  \
             -p 9301:8301 \
             -p 9301:8301/udp \
             -p 9302:8302/udp \
             -p 9302:8302 \
             -p 9400:8400 \
             -p 9500:8500 \
             -p 9600:8600 \
             -h node2 \
             consul agent -server -bind=172.17.0.3 \
             -join=192.168.127.128 -node-id=$(uuidgen | awk '{print tolower($0)}') \
             -node=node2 \
             -data-dir=/tmp/data-dir -client 0.0.0.0 -ui
   docker run -d --name=node3 --restart=always \
             -e 'CONSUL_LOCAL_CONFIG={"skip_leave_on_interrupt": true}' \
             -p 10300:8300  \
             -p 10301:8301 \
             -p 10301:8301/udp \
             -p 10302:8302/udp \
             -p 10302:8302 \
             -p 10400:8400 \
             -p 10500:8500 \
             -p 10600:8600 \
             -h node2 \
             consul agent -server -bind=172.17.0.4 \
             -join=192.168.127.128 -node-id=$(uuidgen | awk '{print tolower($0)}') \
             -node=node3 \
             -data-dir=/tmp/data-dir -client 0.0.0.0 -ui
 //启动client
    docker run -d --name=consulclient --restart=always -e 'CONSUL_LOCAL_CONFIG={"leave_on_terminate":true}' 
 ```
   - –net=host docker参数, 使得docker容器越过了netnamespace的隔离，免去手动指定端口映射的步骤

    - -server consul支持以server或client的模式运行, server是服务发现模块的核心, client主要用于转发请求

    - -advertise 将本机私有IP传递到consul

    - -bootstrap-expect 指定consul将等待几个节点连通，成为一个完整的集群

    - -retry-join 指定要加入的consul节点地址，失败会重试, 可多次指定不同的地址

    - -client consul绑定在哪个client地址上，这个地址提供HTTP、DNS、RPC等服务，默认是127.0.0.1

    - -bind 该地址用来在集群内部的通讯，集群内的所有节点到地址都必须是可达的，默认是0.0.0.0

    - -allow_stale 设置为true, 表明可以从consul集群的任一server节点获取dns信息, false则表明每次请求都会经过consul server leader

    - --name DOCKER容器的名称

    - -client 0.0.0.0 表示任何地址可以访问。

    - -ui  提供图形化的界面。

-advertise：通知展现地址用来改变我们给集群中的其他节点展现的地址，默认情况下-bind地址就是展现地址，然而也存在一些路由地址是不能受约束的，这时候会激活一个不同的地址来供应，如果这个地址不能路由，这个路由将不能被加入集群

-bootstrap：用来控制一个server是否在bootstrap模式，在一个datacenter中只能有一个server处于bootstrap模式，当一个server处于bootstrap模式时，可以自己选举为raft leader，在一个节点的模式下这种方式很重要，否则在集群中的一致性不能保证，不推荐在集群中应用这个标识

-bootstrap-expect：在一个datacenter中期望提供的server节点数目，当该值提供的时候，consul一直等到达到指定sever数目的时候才会引导整个集群，该标记不能和bootstrap公用（推荐使用的方式）

-bind：该地址用来在集群内部的通讯，集群内的所有节点到地址都必须是可达的，默认是0.0.0.0，这意味着Consuelo会使用第一个可用的私有IP地址，Consul可以使用TCP和UDP并且可以使用共同的端口，如果存在防火墙，这两者协议必须是允许的。

-client：consul绑定在哪个client地址上，这个地址提供HTTP、DNS、RPC等服务，默认是127.0.0.1，只允许回路连接。RPC地址用于Consul命令，比如Consul members可以查询当前运行的Consul代理

-config-file：明确的指定要加载哪个配置文件，文件下的所有配置会合并在一起进行加载

-config-dir：配置文件目录，里面所有以.json结尾的文件都会被加载

-data-dir：提供一个目录用来存放agent的状态，所有的agent允许都需要该目录，该目录必须是稳定的，系统重启后都继续存在

-dc：该标记控制agent运行的datacenter的名称，默认是dc1

-encrypt：指定secret key，使consul在通讯时进行加密，key可以通过consul keygen生成，同一个集群中的节点必须使用相同的key

-http-port:HTTP API侦听端口,默认端口8500，可以在环境变量中进行设置，非常有用，可以用于与Consul进行通讯

-join：加入一个已经启动的agent的ip地址，可以多次指定多个agent的地址。如果consul不能加入任何指定的地址中，则agent会启动失败，默认agent启动时不会加入任何节点。

-retry-join：和join类似，但是允许你在第一次失败后进行尝试。

-retry-interval：两次join之间的时间间隔，默认是30s

-retry-max：尝试重复join的次数，默认是0，也就是无限次尝试

-log-level：consul agent启动后显示的日志信息级别。默认是info，可选：trace、debug、info、warn、err。跟踪 调试 详情 警告 错误，可以通过Consul monitor使用任何级别，也可以通过重启加载新的配置级别

-node：节点在集群中的名称，在一个集群中必须是唯一的，默认是该节点的主机名（代表一个机器）

-pid-file:提供一个路径来存放pid文件，可以使用该文件进行SIGINT/SIGHUP(关闭/更新)agent

-protocol：consul使用的协议版本 consul -v

-rejoin：使consul忽略先前的离开，在再次启动后仍旧尝试加入集群中。

-server：定义agent运行在server模式还是Client模式，提供时即为Server端，每个集群至少有一个server并且每台机器上不要超过5个dataceter.所有服务器采用一致性算法Raft保证数据一致,确保在故障的情况下的可用性。

-syslog：开启系统日志功能，只在linux/osx上生效

-ui-dir:提供存放web ui资源的路径，该目录必须是可读的

参数列表	参数的含义和使用场景说明
advertise	通知展现地址用来改变我们给集群中的其他节点展现的地址，一般情况下-bind地址就是展现地址
bootstrap	用来控制一个server是否在bootstrap模式，在一个datacenter中只能有一个server处于bootstrap模式，当一个server处于bootstrap模式时，可以自己选举为raft leader
bootstrap-expect	在一个datacenter中期望提供的server节点数目，当该值提供的时候，consul一直等到达到指定sever数目的时候才会引导整个集群，该标记不能和bootstrap共用
bind	该地址用来在集群内部的通讯IP地址，集群内的所有节点到地址都必须是可达的，默认是0.0.0.0
client	consul绑定在哪个client地址上，这个地址提供HTTP、DNS、RPC等服务，默认是127.0.0.1
config-file	明确的指定要加载哪个配置文件
config-dir	配置文件目录，里面所有以.json结尾的文件都会被加载
data-dir	提供一个目录用来存放agent的状态，所有的agent允许都需要该目录，该目录必须是稳定的，系统重启后都继续存在
dc	该标记控制agent允许的datacenter的名称，默认是dc1
encrypt	指定secret key，使consul在通讯时进行加密，key可以通过consul keygen生成，同一个集群中的节点必须使用相同的key
join	加入一个已经启动的agent的ip地址，可以多次指定多个agent的地址。如果consul不能加入任何指定的地址中，则agent会启动失败，默认agent启动时不会加入任何节点
retry-interval	两次join之间的时间间隔，默认是30s
retry-max	尝试重复join的次数，默认是0，也就是无限次尝试
log-level	consul agent启动后显示的日志信息级别。默认是info，可选：trace、debug、info、warn、err
node	节点在集群中的名称，在一个集群中必须是唯一的，默认是该节点的主机名
protocol	consul使用的协议版本
rejoin	使consul忽略先前的离开，在再次启动后仍旧尝试加入集群中
server	定义agent运行在server模式，每个集群至少有一个server，建议每个集群的server不要超过5个
syslog	开启系统日志功能，只在linux/osx上生效
pid-file	提供一个路径来存放pid文件，可以使用该文件进行SIGINT/SIGHUP(关闭/更新)agent