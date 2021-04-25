# docker 部署go项目

## 编写简单的go代码,创建项目，我是用go module 模式，所以要先执行go mod init gotest

```goland
    package main

    import (
        "log"
        "net/http"
    )

    func main() {
        http.HandleFunc("/", func(resp http.ResponseWriter, req *http.Request) {
            resp.Write([]byte("hello word"))
        })
        if err := http.ListenAndServe(":8888", nil); err != nil {
            log.Fatal("启动错误", err.Error())
        }
    }
```

## 打包,创建xxx.bat文件编写内容

```cmd
    set GOOS=linux
    set GOARCH=amd64
    go build 
```

## 编写Dockerfile 特别注意大小写，我用小写好像有问题

```dockerfile
    FROM centos
    WORKDIR /build
    COPY  ./build /build
    EXPOSE 8888
    ENTRYPOINT [ "./gotest" ]
```

## 或者golang容器直接创建，直接编译,这个会将代码也放上去，所以相对而言比较大

```dockerfile
    FROM golang:latest
    WORKDIR /Users/admin/go_mod/test
    ADD . /Users/admin/go_mod/test
    RUN set GOOS=linux
    RUN set GOARCH=amd64
    RUN go build .
    EXPOSE 8888
    ENTRYPOINT ["./gotest"]
```

## 同样可以先打好包再编译

```dockerfile
    FROM golang:alpine
    WORKDIR /Users/admin/go_mod/test
    ADD ./build /Users/admin/go_mod/test
    EXPOSE 8888
    ENTRYPOINT ["./gotest"]
```

## 特别注意，ENTRYPOINT必须./表示当前路径下，否则会no found

## 创建镜像

docker build -t 名称:版本 . //记住后面有个.

## 创建容器，必须指定端口号，否则好像无法暴露

docker run -it -d --name test -p 8080:8888 gotest:v1

## 使用make做步骤管理

1. 安装mingw64下载很慢的,建议翻墙，然后将mingw64/bin放入path中，在mingw64/bin下有个mingw64_make.exe重命名为make既可

2. 创建Makefile文件

```makefile
    .PHONY: build
    build:
        set GOOS=linux
        set GOARCH=amd64
        go build -o build/gotest
    .PHONY: docker
    docker:
        docker build -t gotest:v1 .
    .PHONY:run
    run:
        docker run --name test1 -it -d -p 8888:8888  gotest:v1
```

3. 按步骤执行， make build=>make docker =>make run


## 使用alpine镜像吧，alpine>centos>golang:alpine:golang:latest