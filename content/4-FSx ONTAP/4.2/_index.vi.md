---
title: "EFS CSI Driver"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 4.2 </b>"
---


Trước khi đi sâu vào phần này, hãy đảm bảo bạn đã làm quen với các đối tượng lưu trữ trong Kubernetes (volumes, persistent volumes (PV), persistent volume claim (PVC), cấp phát động và lưu trữ tạm thời) chúng tôi đã giới thiệu ở [đầu workshop này](../../1-Introduce/1.10-EKS%20Storage).

Các trình điều khiển CSI cho Amazon FSx for NetApp ONTAP (Amazon FSxN ONTAP) giúp chạy các ứng dụng container dựa trên trạng thái. Amazon FSxN ONTAP CSI cung cấp giao diện lưu trữ container cho phép các cụm Kubernetes chạy trên AWS quản lý vọng đời của các hệ tập tin FSxN ONTAP.

Để cấp phát động hệ tập tin Amazon FSxN ONTAP trên cụm EKS, ta cần xác nhận [trình điều khiển Amazon FSxN ONTAP CSI](https://github.com/NetApp/trident) đã được cài đặt. Trình điều khiển Amazon FSxN ONTAP CSI triển khai các thông số cho trình điều phối container nhằm quản lý vòng đời hệ tập tin Amazon FSxN ONTAP.

Như một phần môi trường workshop, cụm EKS đã được cài đặt sẵn trình điều khiển Amazon FSxN ONTAP CSI. Ta có thể xác nhận điều này bằng lệnh sau:

```bash
$ kubectl get pods -n trident
NAME                                  READY   STATUS    RESTARTS   AGE
trident-controller-68f86749df-tr9nw   6/6     Running   0          25m
trident-node-linux-7wkg9              2/2     Running   0          25m
trident-node-linux-9g6w4              2/2     Running   0          25m
trident-node-linux-vpvnh              2/2     Running   0          25m
trident-operator-56fb7f67c4-vws4m     1/1     Running   0          29m
```
Trình điều khiển Amazon FSxN ONTAP CSI hỗ trợ cấp phát động và tĩnh. Hiện tại, việc cấp phát động sẽ tạo một điểm truy cập cho mỗi PersistentVolume. Điều này có nghĩa là hệ tập tin AWS EFS trước tiên phải được tạo thủ công trên AWS và phải được cấp phát làm đầu vào cho tham số StorageClass. Để cấp phát tĩnh, trước tiên, hệ thống tệp AWS EFS cần được tạo thủ công trên AWS, sau đó có thể được gắn vào container bằng trình điều khiển dưới dạng một volume.

Môi trường workshop cũng bao gồm một hệ tập tin FSxN ONTAP, một máy ảo lưu trữ (_Storage Virtual Machine (SVM)_) và các Security Group được cung cấp sẵn với quy tắc đầu vào cho phép lưu lượng NFS đi vào các pod. Thông tin về hệ tập tin FSxN ONTAP có thể được truy vấn từ lệnh AWS CLI sau:

```bash
$ aws fsx describe-file-systems --file-system-id $FSXN_ID
```

Bây giờ, chúng ta cần tạo một đối tượng `TridentBackendConfig` được cấu hình để sử dụng hệ tập tin FSxN ONTAP được cung cấp sẵn như một phần của cơ sở hạ tầng trong workshop này.

Chúng ta sẽ dùng Kustomize để tạo phần backend và lấy biến môi trường `FSN_IP` từ tham số `managementLIF` trong cấu hình đối tượng lớp lưu trữ:

```file
manifests/modules/fundamentals/storage/fsxn/backend/fsxn-backend-nas.yaml
```
Chúng ta áp dụng kustomization:

```bash
$ kubectl kustomize ~/environment/eks-workshop/modules/fundamentals/storage/fsxn/backend \
  | envsubst | kubectl apply -f-
secret/backend-fsxn-ontap-nas-secret created
tridentbackendconfig.trident.netapp.io/backend-fsxn-ontap-nas created
```

Bây giờ ta sẽ chạy lện sau để đảm bảo thành phần `TridentBackendConfig` đã được tạo:

```bash
$ kubectl get tbc -n trident
NAME                     BACKEND NAME          BACKEND UUID                           PHASE   STATUS
backend-fsxn-ontap-nas   backend-fsxn-ontap-   61a731e0-2f3c-4df9-9e49-5fc120e8247c   Bound   Success
```

Bây giờ t cần tạo một đối tượng [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes/) (lớp lưu trữ) bằng Kustomize:

```file
manifests/modules/fundamentals/storage/fsxn/storageclass/fsxnstorageclass.yaml
```

Áp dụng Kuztomization bằng lệnh sau:

```bash
$ kubectl apply -k ~/environment/eks-workshop/modules/fundamentals/storage/fsxn/storageclass/
storageclass.storage.k8s.io/fsxn-sc-nfs created
```

Bây giờ chúng ta sẽ miêu tả StorageClass bằng các lệnh dưới đây. Lưu ý rằng chúng ta sử dụng nhà cung cấp `csi.trident.netapp.io` và chế độ cấp phát là `ontap-nas`.

```bash
$ kubectl get storageclass
NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
fsxn-sc-nfs     csi.trident.netapp.io   Delete          Immediate              true                   44s

$ kubectl describe sc fsxn-sc-nfs
Name:            fsxn-sc-nfs
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"allowVolumeExpansion":true,"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"fsxn-sc-nfs"},"parameters":{"backendType":"ontap-nas"},"provisioner":"csi.trident.netapp.io"}

Provisioner:           csi.trident.netapp.io
Parameters:            backendType=ontap-nas
AllowVolumeExpansion:  True
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     Immediate
Events:                <none>
```

Giờ đây chúng ta đã hiểu rõ hơn về EKS StorageClass và FSxN CSI driver. Ở phần kế tiếp, chúng ta sẽ tập trung vào việc chỉnh sửa tài nguyên Microservice để tận dụng FSxN `StorageClass` nhờ Cấp phát động Kubernete và `PersistentVolume` để lưu trữ ảnh minh họa sản phẩm.