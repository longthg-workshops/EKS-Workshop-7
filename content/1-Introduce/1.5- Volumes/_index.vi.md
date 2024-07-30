---
title: "Volume"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: "<b> 1.5 </b>"
---

#### Tích Hợp Vùng Lưu Trữ (Volumes)

Trong phần này, chúng ta sẽ xem xét về **Vùng Lưu Trữ (Volumes)**

- Chúng ta đã thảo luận về lưu trữ Docker, Nếu chúng ta không gắn volume vào container runtime, tất cả dữ liệu sẽ mất khi container bị hủy. Vì vậy, chúng ta cần lưu trữ dữ liệu trong container Docker bằng cách đính kèm volume cho các container khi chúng được tạo ra.
- Dữ liệu được container xử lý được đặt trong volume giúp lưu trữ lâu dài. Nhờ đó, ngay cả khi container bị xóa, dữ liệu vẫn tồn tại trong volume.

- Trong Kubernetes, các POD được tạo ra có tính chất tạm thời. Khi một POD được tạo ra để xử lý dữ liệu và sau đó bị xóa, dữ liệu được nó xử lý cũng bị xóa đi.
- Ví dụ, chúng ta tạo ra một POD đơn giản với chức năng tạo ra một số ngẫu nhiên từ 1 đến 100 và ghi nó vào một tệp tại `/opt/number.out` nhằm lưu trữ vào volume.
- Chúng ta tạo ra một volume cho việc đó. Trong trường hợp này, một đường dẫn `/data` trên máy chủ được chỉ định. Các tệp được lưu trữ trong thư mục data trên nút của tôi. Chúng ta sử dụng trường volumeMounts trong mỗi container để gắn volume dữ liệu vào thư mục `/opt` bên trong container. Số ngẫu nhiên sẽ được ghi vào `/opt` mount bên trong container, đó cũng là volume dữ liệu nằm trong thư mục `/data` trên máy chủ thật. Khi POD bị xóa, tệp chứa số ngẫu nhiên vẫn còn tồn tại trên máy chủ.

![EKS](/images/part1/1-5/00014.png?featherlight=false&width=90pc)


#### Các Lựa Chọn Lưu Trữ volume

- Trong các volume, loại volume hostPath tốt trong trường hợp node đơn lẻ, song không được khuyến nghị sử dụng trong các cụm nhiều node.
- Trong Kubernetes, nó hỗ trợ một số loại giải pháp lưu trữ tiêu chuẩn như NFS, GlusterFS, CephFS hoặc các giải pháp đám mây công cộng như AWS EBS, Azure Disk hoặc Google's Persistent Disk.

![EKS](/images/part1/1-5/00015.png?featherlight=false&width=90pc)



```
volumes:
- name: data-volume
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```

#### Tài Liệu Tham Khảo volume Kubernetes

- https://kubernetes.io/docs/concepts/storage/volumes/
- https://kubernetes.io/docs/tasks/configure-pod-container/configure-volume-storage/
- https://unofficial-kubernetes.readthedocs.io/en/latest/concepts/storage/volumes/
- https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#volume-v1-core