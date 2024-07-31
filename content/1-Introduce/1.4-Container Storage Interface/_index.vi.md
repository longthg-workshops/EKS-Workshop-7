---
title: "Các giao diện container"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 1.4 </b>"
---

#### Giao diện Lưu trữ Container

Trong phần này, chúng ta sẽ tìm hiểu về **Giao diện Lưu trữ Container**.

#### Giao diện Thực thi Container

- Kubernetes sử dụng Docker engine để vận hành các container. Toàn toàn bộ mã để làm việc với Docker được nhúng trong mã nguồn Kubernetes. Bên cạnh đó, còn có các engine vận hành container khác, như _containerd_, _rkt_ và _CRI-O_.
- Giao diện Thực thi Container là một tiêu chuẩn xác định cách một giải pháp điều phối như Kubernetes giao tiếp với các container engine như Docker. Để phát triển bất kỳ giao diện thực thi container mới nào, ta chỉ cần tuân thủ các tiêu chuẩn CRI.

![EKS](/EKS-Workshop-7/images/part1/1-4/00011.png?featherlight=false&width=90pc)

#### Giao diện Mạng Container

- Để hỗ trợ các giải pháp mạng khác nhau, giao diện mạng container được ra mắt. Bất kỳ nhà cung cấp mạng mới nào cũng có thể đơn giản phát triển plugin của họ dựa trên các tiêu chuẩn CNI và làm cho giải pháp của họ hoạt động với Kubernetes.

![EKS](/EKS-Workshop-7/images/part1/1-4/00012.png?featherlight=false&width=90pc)

#### Giao diện Lưu trữ Container

- Giao diện lưu trữ container được phát triển để hỗ trợ nhiều giải pháp lưu trữ. Với CSI, bạn hiện có thể viết các trình điều khiển của riêng mình cho lưu trữ của riêng bạn để hoạt động với Kubernetes. Portworx, Amazon EBS, Azure Managed Disk, GlusterFS, vv.
- CSI không phải là một tiêu chuẩn cụ thể của Kubernetes. Nó được thiết kế như một tiêu chuẩn phổ quát, cho phép bất kỳ công cụ điều phối container nào làm việc với bất kỳ nhà cung cấp lưu trữ nào có plugin được hỗ trợ. Kubernetes, Cloud Foundry và Mesos đều tham gia với CSI.
- Nó xác định một tập hợp các RPC hoặc cuộc gọi thủ tục từ xa sẽ được gọi bởi trình điều phối container. Các trình điều khiển lưu trữ phải triển khai các RPC này.

![EKS](/EKS-Workshop-7/images/part1/1-4/00013.png?featherlight=false&width=90pc)

#### Tài liệu tham khảo

- https://github.com/container-storage-interface/spec
- https://kubernetes-csi.github.io/docs/
- http://mesos.apache.org/documentation/latest/csi/
- https://www.nomadproject.io/docs/internals/plugins/csi#volume-lifecycle