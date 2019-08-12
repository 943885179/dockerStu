>下载[docker](https://hub.docker.com/editions/community/docker-ce-desktop-windows)
>安装会自动启动Hyper-V,但是如果系统虚拟化未开启的话还是要进BIOS开启
 - (联想)开机长按F2=>Configuration=>Intel Virtual Technology 改为Enabled
>可以直接启动了，当然也可以配置下存放地址等

>cmd 查看是否安装成功`docker version`

>常用命令
 - docker version 查看版本信息
 - docker search [名称] 搜索镜像 例子:docer search tomcat
 - docker pull [名称]/[名称:版本] 拉取镜像
 - docker run  -it --rm ‐d -p 8888:8080 ‐‐name  名称  [image镜像名称] 根据镜像启动容器 
   - -d 后台运行 成功启动容器后输出容器的完整ID.
   - –name：给新创建的容器命名，此处命名为ly-mysql
   - -e：配置信息
   - -p 端口映射 将主机的端口映射到容器的一个端口 主机端口:容器内部的端口
   - -it         #  是-i  和 -t的简写， 表示以交互式的方式运行容器，加上-d表示后台运行，这里为了截图输出启动日志我用了-it，也可以用-d，再用"docker logs 容器名"命令输出日志
   - --rm        #当容器被停止时自动删除容器
   - -p 8888:80  #80是为容器中的tomcat设置的端口， 这里表示将80映射到宿主机8888端口， 如果只写-p 80  容器会随机取值32768~61000中较大的端口号来映射到80端口上
   - v     # 将tomcat中的usr/local/tomcat/webapps目录映射到宿主机当前目录的webapps目录，后面更新jar包直接扔到被映射的宿主机目录中即可
   - 例子：docker run -it --rm -p 8888:8080 -v $PWD/webapps:/usr/local/tomcat/webapps tomcat:8.0[参数说明](https://blog.csdn.net/qq_31807569/article/details/90046287)
 - docker ps 查看运行中的容器
 - docker stop 容器Id 停止容器
 - docker start 容器Id 启动容器
 - docker ps -a 查看所有容器
 - docker rm 容器Id 删除容器
 - docker images 查看镜像列表
 - docker logs container‐name/container‐id 查看容器日志
 - docker rmi imageId
 - docker exec -it [容器名称]  [文件地址] 进入容器，输入`exit`退出 `docker exec -it a21af3387bef  /bin/bash`
 - 部署tomcat项目:docker cp xxx.war tomcat8:/usr/local/tomcat/webapps/
 - docker inspect 容器名称  查看容器所有状态信息
 - docker inspect --format='{{.NetworkSettings.IPAddress}}' ID/NAMES 查看容器的Ip 或者用`docker exec -it ID/NAMES ip addr `
 - docker inspect --format '{{.Name}} {{.State.Running}}' NAMES 容器运行状态
 - docker top NAMES 查看进程
 - docker port ID/NAMES 查看端口
 - docker rmi 镜像 删除镜像
 - docker build -t [自己镜像的名字] . 构建镜像进入Dockerfile所在目录，运行命令 docker build -t mytomcat . (注意最后有个点用来表示当前目录，初次构建速度会比较慢，需要多等一会。)
 - docker commit 将容器打包成镜像
   - -a, --author string    作者（例如，“John Hannibal Smith hannibal@a-team.com”）
   - -c, --change list      应用dockerfile指令来创建图像
   - -m, --message string   提交信息
   - -p, --pause            提交期间暂停容器（默认为true）
   - docker tag 打标签，
   - docker tag fce289e99 hello-world:v1  使用docker tag使用镜像ID重命名
   - docker tag hello-world:latest hello-world:v2 使用docker tag使用镜像tag重命名
   - docker tag 本地镜像名称 用户名/仓库:版本标签 打上标签，后push
   - docker push 用户名/仓库:版本标签 上传到dockerhub