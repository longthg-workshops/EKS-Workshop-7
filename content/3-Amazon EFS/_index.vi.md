---
title: "Amazon EFS"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 3. </b>"
---

Tham khảo Workshop đầu tiên để tạo cụm EKS cùng các tài nguyên trước khi triển khai môi trường cho phần này. Nếu đã hoàn thành, bạn có thể tiếp tục phần này.

#### Chuẩn bị môi trường cho phần này:

```bash timeout=300 wait=30
$ prepare-environment fundamentals/storage/efs
```

Điều này sẽ thực hiện các thay đổi sau vào môi trường thí nghiệm của bạn:

- _Tạo một vai trò IAM cho trình điều khiển CSI của Amazon EFS_
- _Tạo một hệ thống tập tin Amazon EFS_

Bạn có thể xem Terraform áp dụng các thay đổi này [tại đây](https://github.com/aws-samples/eks-workshop-v2/tree/stable/manifests/modules/fundamentals/storage/ebs/.workshop/terraform).

[Amazon Elastic File System](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html) là một hệ thống tập tin linh hoạt, không cần máy chủ, đơn giản và tự động quên dành cho việc sử dụng với dịch vụ Đám mây AWS và các tài nguyên trên cơ sở. Nó được xây dựng để tự động mở rộng theo yêu cầu đến petabyte mà không làm gián đoạn các ứng dụng, tự động mở rộng và thu nhỏ khi bạn thêm và loại bỏ các tệp, loại bỏ nhu cầu về việc cấp và quản lý dung lượng để chứa sự tăng trưởng.

Trong bài thực hành này, chúng ta sẽ tìm hiểu về các khái niệm sau:

- Triển khai dịch vụ microservices tài sản
- Trình điều khiển CSI của EFS
- Cung cấp động bằng cách sử dụng EFS và một triển khai Kubernetes