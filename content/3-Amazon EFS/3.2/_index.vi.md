---
title: "EFS CSI Driver"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 3.2 </b>"
---

#### EFS CSI Driver

Trước khi chúng ta đào sâu vào phần này, hãy đảm bảo bạn đã làm quen với các đối tượng lưu trữ Kubernetes (thể hiện bằng các khối lưu trữ, khối lưu trữ bền (PV), yêu cầu khối lưu trữ bền (PVC), cấp phát động và lưu trữ tạm thời) đã được giới thiệu trong [Phần Lưu trữ](../index.md).

Trình điều khiển Giao diện Lưu trữ Container Amazon Elastic File System (CSI) (https://github.com/kubernetes-sigs/aws-efs-csi-driver) giúp bạn chạy các ứng dụng container có trạng thái. Trình điều khiển Giao diện Lưu trữ Container Amazon Elastic File System (CSI) cung cấp một giao diện CSI cho phép các cụm Kubernetes chạy trên AWS quản lý vòng đời của các hệ thống tệp Amazon EFS.

Để tận dụng hệ thống tệp Amazon EFS với cấp phát động trên cụm EKS của chúng tôi, chúng ta cần xác nhận rằng chúng ta đã cài đặt Trình điều khiển CSI EFS. Trình điều khiển Giao diện Lưu trữ Container Amazon Elastic File System (CSI) (https://github.com/kubernetes-sigs/aws-efs-csi-driver) triển khai quy định CSI để các bộ triển khai container quản lý vòng đời của các hệ thống tệp Amazon EFS.

Để cải thiện bảo mật và giảm lượng công việc, bạn có thể quản lý trình điều khiển CSI EFS của Amazon như một phần mở rộng của Amazon EKS. Vai trò IAM cần thiết cho phần mở rộng đã được tạo ra cho chúng ta để chúng ta có thể tiến hành cài đặt phần mở rộng:

```bash timeout=300 wait=60
$ aws eks create-addon --cluster-name $EKS_CLUSTER_NAME --addon-name aws-efs-csi-driver \
  --service-account-role-arn $EFS_CSI_ADDON_ROLE
$ aws eks wait addon-active --cluster-name $EKS_CLUSTER_NAME --addon-name aws-efs-csi-driver
```

Bây giờ chúng ta có thể xem những gì đã được tạo ra trong cụm EKS của chúng ta thông qua phần mở rộng. Ví dụ, một DaemonSet sẽ chạy một pod trên mỗi node trong cụm của chúng ta:

```bash
$ kubectl get daemonset efs-csi-node -n kube-system
NAME           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
efs-csi-node   3         3         3       3            3           kubernetes.io/os=linux        47s
```

Trình điều khiển CSI EFS hỗ trợ cấp phát động và tĩnh. Hiện tại, cấp phát động tạo một điểm truy cập cho mỗi PersistentVolume. Điều này có nghĩa là một hệ thống tệp Amazon EFS phải được tạo ra thủ công trên AWS trước và cần được cung cấp như một đầu vào cho tham số StorageClass. Đối với cấp phát tĩnh, hệ thống tệp Amazon EFS cần được tạo ra thủ công trên AWS trước. Sau đó, nó có thể được gắn vào bên trong một container như một khối lưu trữ sử dụng trình điều khiển.

Chúng tôi đã triển khai một hệ thống tệp EFS, các mục tiêu gắn kết và nhóm bảo mật cần thiết đã được triển khai trước với một quy tắc inbound cho phép lưu lượng NFS đến cho các điểm gắn kết Amazon EFS của bạn. Hãy lấy một số thông tin về nó sẽ được sử dụng sau này:

```bash
$ export EFS_ID=$(aws efs describe-file-systems --query "FileSystems[?Name=='$EKS_CLUSTER_NAME-efs-assets'] | [0].FileSystemId" --output text)
$ echo $EFS_ID
fs-061cb5c5ed841a6b0
```

Bây giờ, chúng ta sẽ cần tạo một đối tượng StorageClass được cấu hình để sử dụng hệ thống tệp EFS đã triển khai trước như một phần của cơ sở hạ tầng workshop này và sử dụng [EFS Access points](https://docs.aws.amazon.com/efs/latest/ug/efs-access-points.html) trong chế độ triển khai.

Chúng ta sẽ sử dụng Kustomize để tạo cho chúng ta lớp lưu trữ và để nhập biến môi trường `EFS_ID` vào giá trị tham số `filesystemid` trong cấu hình của đối tượng lớp lưu trữ:

```file
manifests/modules/fundamentals/storage/efs/storageclass/efsstorageclass.yaml
```

Hãy áp dụng kustomization này:

```bash
$ kubectl kustomize ~/environment/eks-workshop/modules/fundamentals/storage/efs/storageclass \
  | envsubst | kubectl apply -f-
storageclass.storage.k8s.io/efs-sc created
```

Bây giờ chúng ta sẽ lấy và mô tả StorageClass bằng các lệnh dưới đây. Chú ý rằng người cung cấp được sử dụng là trình điều khiển EFS CSI và chế độ triển khai là điểm truy cập EFS và ID của hệ thống tệp như đã xuất trong biến môi trường `EFS_ID`.

```bash
$ kubectl get storageclass
NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
efs-sc          efs.csi.aws.com         Delete          Immediate              false                  8m29s
$ kubectl describe sc efs-sc
Name:            efs-sc
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"efs-sc"},"parameters":{"directoryPerms":"700","fileSystemId":"fs-061cb5c5ed841a6b0","provisioningMode":"efs-ap"},"provisioner":"efs.csi.aws.com"}

Provisioner:           efs.csi.aws.com
Parameters:            directoryPerms=700,fileSystemId=fs-061cb5c5ed841a6b0,provisioningMode=efs-ap
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     Immediate
Events:                <none>
```

Bây giờ chúng ta đã hiểu rõ hơn về StorageClass của EKS và trình điều khiển CSI EFS. Trên trang tiếp theo, chúng tôi sẽ tập trung vào việc sửa đổi dịch vụ microservice tài sản để tận dụng `StorageClass` EFS bằng cách sử dụng cung cấp dựng dựa trên thể tích động Kubernetes và một PersistentVolume để lưu trữ hình ảnh sản phẩm.