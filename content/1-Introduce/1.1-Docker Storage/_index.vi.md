---
title: "Lưu trữ trong Docker"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 1.1 </b>"
---

#### Giới thiệu về Lưu trữ Docker
  
Trong phần này, chúng ta sẽ tìm hiểu về **lưu trữ Docker**.

- Để hiểu lưu trữ trong các công cụ điều phối container như **Kubernetes**, điều quan trọng nhất là hiểu rõ cách lưu trữ hoạt động với các container. Hiểu cách lưu trữ hoạt động với **Docker** và nắm chắc tất cả các khái niệm cơ bản sẽ giúp việc hiểu cách nó hoạt động trong **Kubernetes** dễ dàng hơn nhiều sau này.

- Nếu bạn mới bắt đầu với **Docker**, bạn có thể học một số khái niệm cơ bản của docker từ khóa học  [Docker cho người mới bắt đầu](https://kodekloud.com/courses/docker-for-the-absolute-beginner/).

#### Lưu trữ Docker

- Có hai khái niệm liên quan đến Docker, đó là các trình điều khiển lưu trữ (Volume Driver) và các plugin trình điều khiển thư mục.

![EKS](/EKS-Workshop-7/images/part1/1-1/0001.png?featherlight=false&width=90pc)

#### Trước tiên chúng ta sẽ thảo luận về các trình điều khiển lưu trữ.

#### Tài liệu Tham khảo Docker

- https://docs.docker.com/storage/storagedriver/
- https://docs.docker.com/storage/volumes/