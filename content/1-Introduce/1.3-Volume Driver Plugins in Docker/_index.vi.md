---
title: "Plugin Volume Driver trong Docker"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 1.3 </b>"
---

#### Plugin Volume Driver (PVD) trong Docker

Trong phần này, chúng ta sẽ tìm hiểu về **PVD trong Docker**

- Chúng ta đã thảo luận về các driver lưu trữ. Các driver lưu trữ giúp quản lý lưu trữ trên image và container.
- Chúng ta đã biết rằng nếu bạn muốn lưu trữ lâu dài, bạn phải tạo ra các container. Các container không được quản lý bởi các driver lưu trữ. Các container được quản lý bởi các plugin driver container. Plugin driver container mặc định là local.
- Plugin container local giúp tạo ra một container trên máy chủ Docker và lưu trữ dữ liệu của nó dưới thư mục `/var/lib/docker/volumes/`.
- Có nhiều plugin driver container khác cho phép bạn tạo ra một container trên các giải pháp của bên thứ ba như lưu trữ tệp Azure Files, DigitalOcean Block Storage, Portworx, Google Compute Persistent Disk, v.v.

![EKS](/images/part1/1-3/0009.png?featherlight=false&width=90pc)

- Khi bạn chạy một Docker container, bạn có thể chọn sử dụng một plugin driver container cụ thể, như RexRay EBS để cung cấp một container từ Amazon EBS. Điều này sẽ tạo ra một container và gắn kết một container từ điện toán đám mây AWS. Khi container kết thúc, dữ liệu của bạn được an toàn trên điện toán đám mây.

```shell
$ docker run -it --name mysql --volume-driver rexray/ebs --mount src=ebs-vol,target=/var/lib/mysql mysql
```

![EKS](/images/part1/1-3/00010.png?featherlight=false&width=90pc)

#### Tài liệu tham khảo Docker

- https://docs.docker.com/engine/extend/legacy_plugins/
- https://github.com/rexray/rexray