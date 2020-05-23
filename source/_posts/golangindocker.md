---
title: Golang应用部署到Docker
date: 2020-05-23T10:39:26+08:00
tags:
 - golang
 - Docker
---

Golang作为一门静态语言运行前必须完成编译，而Python这类动态语言只要在解释器环境下就可以直接运行，所以他们的docker部署的最佳实践方式会略有不同。![](https://i.loli.net/2020/05/23/dzCOVvEkItop5w4.png)
<!-- more -->

## Dockerfile for Golang

这里我先直接给出Goalng的Dockerfile ：

``````dockerfile
FROM golang:alpine AS builder

# Set necessary environmet variables needed for our image
ENV GO111MODULE=on \
    GOPROXY=https://goproxy.cn,direct \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64

# Move to working directory /build
WORKDIR /build

# Copy and download dependency using go mod
COPY go.mod .
COPY go.sum .
RUN go mod download

# Copy the code into the container
COPY . .

# Run test
#RUN go test ./...

# Build the application
RUN go build -o main .

# Move to /dist directory as the place for resulting binary folder
WORKDIR /dist

# Copy binary from build to main folder
RUN cp /build/main .

############################
# STEP 2 build a small image
############################
FROM scratch

COPY --from=builder /dist/main /
COPY ./env/demo.env /env/demo.env
COPY ./logs /logs

# Command to run the executable
ENTRYPOINT ["/main"]
``````

## Dockerfile for Python

因而在我看到一些高star数的开源软件的Dockerfile中，Python程序一般直接在运行在`python:3.7-alpine`的基础镜像之上。比如:

``````python
FROM python:3.7-alpine

ENV PYTHONUNBUFFERED 1

RUN apk update \
  # psycopg2 dependencies
  && apk add --virtual build-deps gcc python3-dev musl-dev \
  && apk add postgresql-dev \
  # Pillow dependencies
  && apk add jpeg-dev zlib-dev freetype-dev lcms2-dev openjpeg-dev tiff-dev tk-dev tcl-dev \
  # CFFI dependencies
  && apk add libffi-dev py-cffi

RUN addgroup -S django \
    && adduser -S -G django django

# Requirements are installed here to ensure they will be cached.
COPY ./requirements /requirements
RUN pip install --no-cache-dir -r /requirements/production.txt \
    && rm -rf /requirements

COPY ./compose/production/django/entrypoint /entrypoint
RUN sed -i 's/\r$//g' /entrypoint
RUN chmod +x /entrypoint
RUN chown django /entrypoint

COPY ./compose/production/django/start /start
RUN sed -i 's/\r$//g' /start
RUN chmod +x /start
RUN chown django /start
COPY ./compose/production/django/celery/worker/start /start-celeryworker
RUN sed -i 's/\r$//g' /start-celeryworker
RUN chmod +x /start-celeryworker
RUN chown django /start-celeryworker

COPY ./compose/production/django/celery/beat/start /start-celerybeat
RUN sed -i 's/\r$//g' /start-celerybeat
RUN chmod +x /start-celerybeat
RUN chown django /start-celerybeat

COPY ./compose/production/django/celery/flower/start /start-flower
RUN sed -i 's/\r$//g' /start-flower
RUN chmod +x /start-flower
COPY . /app

RUN chown -R django /app

USER django

WORKDIR /app

ENTRYPOINT ["/entrypoint"]

``````



## 总结

上面可以看出python的应用一般直接跑在某个python版本的alpine基础环境下的，而Gloang则是在GO的alpine环境下完成编译与测试后，然后通过多阶段构建的策略，最后将编译完成Gloang程序运行在`scratch`镜像中，这样可以减小Golang程序最终的镜像大小，避免浪费资源。