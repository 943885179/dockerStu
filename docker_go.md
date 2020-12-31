# go 部署到docker

## 编写main.go

## 编写Dockerfile文件

```dockerfile
FROM golang
WORKDIR $GOPATH/src/godocker

ADD . $GOPATH/src/godocker

RUN go build main.go

EXPOSE 8080

ENTRYPOINT ["./main"]
```

## 构建 `docker build -t xxx:v1 .`

## 启动容器`docker run -d -p 8080:8080 xxx:v1`

## 使用golang吧太大了使用alpine

```dockerfile
FROM alpine:latest
WORKDIR $GOPATH/src/godocker

ADD . $GOPATH/src/godocker
COPY . .

EXPOSE 8080

ENTRYPOINT ["./main"]
```
