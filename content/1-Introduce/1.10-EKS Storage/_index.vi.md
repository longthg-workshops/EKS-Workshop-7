---
title: "Lưu trữ trên EKS"
date: "`r Sys.Date()`"
weight: 10
chapter: false
pre: "<b> 1.10 </b>"
---

#### Lưu trữ trên EKS

[Bài viết này](https://docs.aws.amazon.com/eks/latest/userguide/storage.html) sẽ cung cấp một cái nhìn tổng quan về cách tích hợp hai dịch vụ Lưu trữ của AWS với cụm EKS của bạn.

Trước khi chúng ta đào sâu vào việc triển khai, dưới đây là một tóm tắt về hai dịch vụ lưu trữ của AWS mà chúng ta sẽ sử dụng và tích hợp với EKS:

- [Amazon Elastic Block Store](https://aws.amazon.com/ebs/) (hỗ trợ EC2): một dịch vụ lưu trữ khối cho phép truy cập trực tiếp từ các EC2 Instance và container đến một ổ lưu trữ được thiết kế cho cả hiệu suất và khối lượng công việc giao dịch ở bất kỳ quy mô nào.
- [Amazon Elastic File System](https://aws.amazon.com/efs/) (hỗ trợ Fargate và EC2): một hệ thống tệp được quản lý toàn phần, có khả năng mở rộng và đàn hồi phù hợp cho phân tích dữ liệu lớn, phục vụ web và quản lý nội dung, phát triển và kiểm thử ứng dụng, quy trình công việc phương tiện và giải trí, sao lưu cơ sở dữ liệu và lưu trữ container. EFS lưu trữ dữ liệu của bạn một cách dự phòng trên nhiều Khu vực Khả dụng (AZ) và cung cấp truy cập độ trễ thấp từ các pod Kubernetes bất kể AZ chúng đang chạy ở.
- [Amazon FSx for NetApp ONTAP](https://aws.amazon.com/fsx/netapp-ontap/) (hỗ trợ EC2): Lưu trữ chia sẻ được quản lý toàn phần, được xây dựng trên hệ thống tệp ONTAP phổ biến của NetApp. FSx cho NetApp ONTAP lưu trữ dữ liệu của bạn một cách dự phòng trên nhiều Khu vực Khả dụng (AZ) và cung cấp truy cập độ trễ thấp từ các pod Kubernetes bất kể AZ chúng đang chạy ở.
- [FSx for Lustre](https://aws.amazon.com/fsx/lustre/) (hỗ trợ EC2): một hệ tập tin được quản lý toàn phần, có hiệu suất cao, được tối ưu hóa cho các khối công việc như học máy, tính toán hiệu suất cao, xử lý video, mô hình tài chính, tự động hóa thiết kế điện tử và phân tích. Với FSx cho Lustre, bạn có thể nhanh chóng tạo ra một hệ thống tệp hiệu suất cao liên kết với kho lưu trữ dữ liệu S3 của bạn và truy cập các đối tượng S3 một cách trong suốt như các tệp. **FSx sẽ được thảo luận trong các module tương lai của workshop này**

Quan trọng hơn cả là nắm vững một số khái niệm về [Lưu trữ Kubernetes](https://kubernetes.io/docs/concepts/storage/):

- [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/): Các tệp trên đĩa trong một container là tạm thời, điều này đặt ra một số vấn đề đối với các ứng dụng phức tạp khi chạy trong các container. Một vấn đề là mất các tệp khi một container gặp sự cố. Kubelet khởi động lại container ở trạng thái sạch. Một vấn đề thứ hai xảy ra khi chia sẻ tệp giữa các container chạy cùng nhau trong một Pod. Trừu tượng hóa khối lưu trữ Kubernetes giải quyết cả hai vấn đề này. Việc quen thuộc với Pods được đề xuất.
- [Ephemeral Volumes](https://kubernetes.io/docs/concepts/storage/ephemeral-volumes/) Được thiết kế để lưu trữ các dữ liệu tạm thời (file đầu vào, cache, bộ nhớ phụ,...). Vì các khối lưu trữ tuân theo vòng đời của Pod, được tạo ra và xóa cùng Pod, các Pod có thể bị dừng và khởi động lại mà không bị giới hạn ở nơi có sẵn một số khối lưu trữ bền.
- [Persistent Volumes (PV)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) là một thành phần của lưu trữ trong một cụm, được cung cấp bởi một quản trị viên hoặc được cung cấp động bằng cách sử dụng Lớp Lưu trữ. Đây là một tài nguyên trong cụm giống như một nút là một tài nguyên của cụm. PVs là plugin khối lưu trữ giống như Volumes, nhưng có vòng đời độc lập với bất kỳ Pod cụ thể nào sử dụng PV. Đối tượng API này ghi lại các chi tiết của việc triển khai lưu trữ, có thể là NFS, iSCSI hoặc hệ thống lưu trữ cụ thể của nhà cung cấp đám mây.
- [Persistent Volume Claim (PVC)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) là yêu cầu lưu trữ từ người dùng. Các yêu cầu này có thể xác định cụ thể kích thước và [chế độ truy cập](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes) (ReadWriteOnce, ReadOnlyMany or ReadWriteMany).
- [Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/) cung cấp phương thức đặc tả lớp lưu trữ cho quản trị viên. Các lớp lưu trữ có thể được gắn/ánh xạ các mức chất lượng dịch vụ khác nhau, chính sách sao lưu, hoặc các chính sách khác được định nghĩa bởi quản trị viên cụm. Kubernetes không đưa ra quan điểm về những gì các lớp lưu trữ đại diện.
- [Dynamic Volume Provisioning (cấp phát động)](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/) cho phép tạo volume theo yêu cầu. Cấp phát động giúp quản trị viên không phải tự liên hệ và yêu cầu các nhà cung cấp, cũng như tạo PersistentVolume một cách thủ công.
