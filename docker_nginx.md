###### 下载nginx `docker pull nginx`
###### 默认运行nginx `docker run -d  --name dockernginx -p 8888:80 nginx`
###### 打开cmd进入你要放nginx的目录创建`mkdir nginx `
- 进入nginx目录 `cd nginx`
- 创建文件conf文件夹 `mkdir conf` 目录里的配置文件将映射为 nginx 容器的配置文件
- 创建logs文件夹 `mkdir logs`目录将映射为 nginx 容器的日志目录。
- 创建www文件夹 `mkdir www`目录将映射为 nginx 容器配置的虚拟目录。
###### 将nginx.conf拷贝到本地 `docker cp dockernginx:/etc/nginx/nginx.conf /nginx/conf`
###### 将本地文件挂载到docker的nginx上 
 ```
  //先删除原来的nginx
  docker stop dockernginx  
  docker rm dockernginx
  //创建新的nginx容器
  docker run --name dockernginx -d -p 8080:80 -v g:/docker/nginx/nginx.conf:/etc/nginx/nginx.conf -v g:/docker/nginx/www:/usr/share/nginx/html -v g:/docker/nginx/logs:/var/log/nginx nginx
 ```
 >常见错误：参照别人的博客解决的[链接](https://blog.csdn.net/weixin_33860528/article/details/93179590)
  ```
  Error response from daemon: OCI runtime create failed: container_linux.go:345: starting container process caused
  这个错误是一开始我cmd cd到本地nginx然后填写瓜子路径使用相对路径报错的，一定要用绝对路径
  ```
###### nginx.conf配置，参照[网址](http://www.360doc.com/content/18/0111/23/25533110_721195668.shtml)
 ```   
    user  nginx; # 定义nginx运行的用户和用户组 user nginx nginx;改为特殊的用户和组
    worker_processes  8; # nginxworker进程数，即处理请求的进程
    # worker_cpu_affinity 0001 0010 0100 1000 0001 00100100 1000; cup亲和力配置，让不同进程使用不同的cpu
    error_log  /var/log/nginx/error.log warn; #全局错误日志定义类型[debug|info|notice|warn|error|crit]
    pid        /var/run/nginx.pid; # 把进程号记录到文件，用于管理nginx进程
    # worker_rlimit_nofile 65535; nginxworker最大打开文件数，可设置为系统优化后的ulimit -HSn的结果
    events { # IO事件模型与worker进程连接数设置
        worker_connections  10240; #单个worker进程最大连接数 [nginx最大连接数=worker连接数*worker进程数] 
    }
    http {
        # server_tokens off; #隐藏响应header和错误通知中的版本号
        include       /etc/nginx/mime.types; #文件拓展名与文件类型映射表
        default_type  application/octet-stream;#默认文件类型
        # server_names_hash_max_size 512; # 服务域名的最大hash表大小
        # server_names_hash_bucket_size 128; # 服务域名的hash表大小
        # 设定访问日志的日志纪律格式
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';

        access_log  /var/log/nginx/access.log  main;

        sendfile        on; # 开启高效文件传输模式，实现内核零拷贝
        #tcp_nopush     on; tcp_nopush参数可以允许把httpresponse header和文件的开始放在一个文件里发布，积极的作用是减少网络保温段的数量
        # tcp_nodelay on; # 激活tcp_nodelay,内核会等待将更多的字节组成一个数据包，从而提高I/O性能
        keepalive_timeout  65; # 连接超时时间，单位秒
        # autoindex off;目录列表访问参数，合适http下载，默认关闭
        # client_header_timeout 15s; # 读取客户端请求头的超时时间
        #gzip  on; #开启gzip压缩功能
        # 反向代理负载均衡设定部分 upstream表示负载服务器池，
        #upstream www.xxx.com{
            #server是服务器节点起始标签，其后为节点地址，可以是域名或ip， weight权重，越大越有概率访问
           # server 127.0.0.1:80 weight=1;
           # server www.xxx.com:8410 weight=1 backup; #vackup 热备
            #ip_hash;
        # }
        include /etc/nginx/conf.d/*.conf;
    }
    server{
        listen 80; #监听端口，也可以是192.168.1.1:80的形式
        # server_name wwww.xxx.com;#域名
        # root html/blog;#站点根目录，
        location /{#默认访问的location标签段
           index index.html|index.aspx:index.jsp; #首页排序
        } 
    }
 ```