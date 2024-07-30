---
title: "Amazon FSx trên NetApp ONTAP"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 4. </b>"
---

Tham khảo Workshop đầu tiên để tạo cụm EKS cùng các tài nguyên trước khi triển khai môi trường cho phần này. Nếu đã hoàn thành, bạn có thể tiếp tục phần này.

#### Chuẩn bị môi trường cho phần này:

```bash timeout=1800 wait=30
$ prepare-environment fundamentals/storage/fsxn
```

[Amazon FSx for NetApp ONTAP](https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/what-is-fsx-ontap.html) (FSxN) là dịch vụ lưu trữ cho phép triển khai và chạy hệ tập tin ONTAP được quản lý toàn phần trên nền tảng đám mây. ONTAP là công nghệ hệ tập tin của NetApp, cung cấp tập hợp năng lực truy cập và quản lý dữ liệu được áp dụng rộng rãi. _Amazon FSx for NetApp ONTAP_ cung cấp sự linh hoạt, đơn giản và khả năng mở rộng của dịch vụ hoàn toàn được AWS quản lý cùng các tính năng, hiệu suất và các API của hệ thống NetApp tại chỗ.

Trong phần này, chúng ta sẽ học các khái niệm sau:

- Triển khai tài nguyên dưới dạng vi dịch vụ
- FSx cho NetApp ONTAP CSI Driver
- Cấp phát động với _FSx for NetApp ONTAP_ và Kubernetes