---
title: "EBS CSI Driver"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 2.2 </b>"
---

#### EBS CSI Driver

Trước khi chúng ta đi sâu vào phần này, hãy đảm bảo bạn đã làm quen với các đối tượng lưu trữ Kubernetes (thư mục, thư mục có sức chứa (PV), yêu cầu thư mục có sức chứa (PVC), cấp phát động và lưu trữ tạm thời) được giới thiệu trong phần chính [Storage](../index.md).

[**emptyDir**](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir) là một ví dụ về các thư mục tạm thời, và hiện tại chúng ta đang sử dụng nó trên MySQL StatefulSet, nhưng chúng ta sẽ làm việc để cập nhật nó trong chương này thành Một Thư Mục Có Sức Chứa (PV) sử dụng Cấp Phát Động.

[Giao diện Lưu trữ Container Kubernetes (CSI)](https://kubernetes-csi.github.io/docs/) giúp bạn chạy các ứng dụng chứa trạng thái. Các trình điều khiển CSI cung cấp một giao diện CSI cho phép các cụm Kubernetes quản lý vòng đời của các thư mục có sức chứa. Amazon EKS giúp bạn chạy các khối công việc có trạng thái một cách dễ dàng hơn bằng cách cung cấp các trình điều khiển CSI cho Amazon EBS.

Để sử dụng các thư mục Amazon EBS với cấp phát động trên cụm EKS của chúng tôi, chúng ta cần xác nhận rằng chúng ta đã cài đặt Trình điều khiển CSI EBS. [Trình điều khiển giao diện lưu trữ Container Elastic Block Store (Amazon EBS)](https://github.com/kubernetes-sigs/aws-ebs-csi-driver) cho phép các cụm dịch vụ Amazon Elastic Kubernetes (Amazon EKS) quản lý vòng đời của các thư mục Amazon EBS cho các thư mục có sức chứa.

Để cải thiện bảo mật và giảm lượng công việc, bạn có thể quản lý Trình điều khiển CSI EBS của Amazon như một tiện ích bổ sung cho Amazon EKS. Vai trò IAM cần thiết bởi tiện ích bổ sung đã được tạo cho chúng ta vì vậy chúng ta có thể tiến hành cài đặt tiện ích bổ sung:

```bash timeout=300 wait=60
$ aws eks create-addon --cluster-name $EKS_CLUSTER_NAME --addon-name aws-ebs-csi-driver \
  --service-account-role-arn $EBS_CSI_ADDON_ROLE
$ aws eks wait addon-active --cluster-name $EKS_CLUSTER_NAME --addon-name aws-ebs-csi-driver
```

Bây giờ chúng ta có thể xem xét những gì đã được tạo ra trong cụm EKS của chúng ta bởi tiện ích bổ sung. Ví dụ, một DaemonSet sẽ chạy một pod trên mỗi nút trong cụm của chúng ta:

```bash
$ kubectl get daemonset ebs-csi-node -n kube-system
NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
ebs-csi-node   3         3         3       3            3           kubernetes.io/os=linux   3d21h
```

Chúng ta cũng đã có đối tượng [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/) được cấu hình sử dụng [loại thư mục GP2 của Amazon EBS](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/general-purpose.html#EBSVolumeTypes_gp2). Chạy lệnh sau để xác nhận:

```bash
$ kubectl get storageclass
NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
gp2 (default)   kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   false                  3d22h
```

Bây giờ chúng ta đã hiểu rõ hơn về EKS Storage và các đối tượng Kubernetes. Trên trang kế tiếp, chúng ta sẽ tập trung vào việc sửa đổi MySQL DB StatefulSet của dịch vụ microcatalog để sử dụng một thư mục lưu trữ khối EBS như lưu trữ có sức chứa cho các tập tin cơ sở dữ liệu sử dụng cấp phát động của Kubernetes.