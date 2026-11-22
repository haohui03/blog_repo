---
title: docker-dockerfile
draft: false
tags:
- docker
- dockerfile
- infra
categories:
- code
url: /zh-cn/1/01/01/category/docker-dockerfile/
authors:
- yehh
---

# Dockerfile 最佳实践 {#dockerfile-最佳实践}

## 分段构建 {#分段构建}

```dockerfile
FROM golang AS build-env
ADD . /go/src/app
WORKDIR /go/src/app
RUN go get -u -v github.com/kardianos/govendor
RUN govendor sync
RUN GOOS=linux GOARCH=386 go build -v -o /go/src/app/app-server

FROM alpine
RUN apk add -U tzdata
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime
COPY --from=build-env /go/src/app/app-server /usr/local/bin/app-server
EXPOSE 8080
CMD [ "app-server" ]
```
