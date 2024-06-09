---
title: "Dockerize môi trường dev cho dự án Golang"
date: "2024-06-06"
# description: "Hướng dẫn thu thập dữ liệu chứng khoán từ công ty X cùng Python và Google Sheet"
tags: ["docker", "golang"]
categories: ["Docker thật là đơn giản", "Golang thật là đơn giản"]
series: ["Docker thật là đơn giản"]
cover:
  image: docker-logo.png
#   caption: ""
ShowToc: true
TocOpen: true
---

Dockerize môi trường dev cho dự án Golang có thể chạy trên mọi hệ điều hành theo cách đơn giản nhất.

<!--more-->

---

## Yêu cầu kiến thức

- Golang
- Docker

---

## Cài đặt

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)

---

## Cấu trúc thư mục

Để đơn giản, mình sẽ sử dụng cấu trúc dưới đây:
```
project/
├── Dockerfile
└── docker-compose.yaml
```
---

## Tiến hành Dockerize

Tuỳ vào dự án, chúng ta sẽ khởi tạo `Dockerfile` và `docker-compose.yaml` khác nhau. Trong bài viết này, mình sẽ khởi tạo duy nhất 1 service dùng chạy code Go và version Go mình sử dụng sẽ là `1.20`.

`Dockerfile`
```dockerfile
ARG VERSION=1.20
FROM golang:${VERSION}
WORKDIR /workspace
COPY . .
```

`docker-compose.yaml`
```yaml
version: "3.8"
services:
  app:
    build: .
    container_name: go-app
    command: /bin/sh -c "while sleep 1000; do :; done"
    volumes:
      - ./:/workspace
    restart: unless-stopped
```

Sau đó, chúng ta mở Terminal/PowerShell, `cd` vào thư mục `project`, chạy lệnh sau để khởi tạo service có trong `docker-compose.yaml`:
```
docker compose up -d
```

Để có thể thực thi code Go, chúng ta cần truy cập vào Terminal của `go-app` container, chạy lệnh sau để mở Terminal:
```
docker exec -e "TERM=xterm-256color" -w /workspace -it go-app bash
``` 
---

## Mở rộng
Từ đây, chúng ta có thể thêm tuỳ ý các service khác (database, message queue,...) vào trong `docker-compose.yaml`.