---
title: "Storage Class"
date: "`r Sys.Date()`"
weight: 9
chapter: false
pre: "<b> 1.9 </b>"
---

#### Lớp Lưu Trữ (Storage Class)

Trong phần này, chúng ta sẽ xem xét về **Lớp Lưu Trữ**

- Chúng ta đã thảo luận về cách tạo PersistentVolume và Yêu Cầu PersistentVolume, cũng như đã thấy cách sử dụng vào không gian Volume của Pod để yêu cầu không gian volume đó.
- Chúng ta đã tạo PersistentVolume, nhưng trước đó nếu đang lấy một volume từ các nhà cung cấp Đám Mây như GCP, AWS, Azure, chúng ta cần phải tạo ổ đĩa trong Google Cloud là một ví dụ.
- Chúng ta cần phải tạo thủ công mỗi khi chúng ta định nghĩa trong tệp định nghĩa Pod. Đây này được gọi là **Cấp Phát Tĩnh**.

#### Cấp Phát Tĩnh

![EKS](/images/part1/1-9/00018.png?featherlight=false&width=90pc)

#### Cấp Phát Động

![EKS](/images/part1/1-9/00019.png?featherlight=false&width=90pc)

- Bây giờ chúng ta có một Lớp Lưu Trữ (StorageClass). Chúng ta sẽ không cần phải định nghĩa PersistentVolume (PV) nữa. PV sẽ được tạo tự động khi một Lớp Lưu Trữ được tạo. Đây được gọi là **Cấp Phát Động**.

```
sc-definition.yaml

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: google-storage
provisioner: kubernetes.io/gce-pd
```

#### Tạo Một Lớp Lưu Trữ

```
$ kubectl create -f sc-definition.yaml
storageclass.storage.k8s.io/google-storage created
```

#### Liệt Kê Các Lớp Lưu Trữ

```
$ kubectl get sc
NAME             PROVISIONER            RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
google-storage   kubernetes.io/gce-pd   Delete          Immediate           false                  20s
```

#### Tạo Yêu Cầu PersistentVolume

```
pvc-definition.yaml

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes: [ "ReadWriteOnce" ]
  storageClassName: google-storage       
  resources:
   requests:
     storage: 500Mi
```
```
$ kubectl create -f pvc-definition.yaml

```
#### Tạo Một Pod

```
pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: frontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: web
  volumes:
    - name: web
      persistentVolumeClaim:
        claimName: myclaim
```
```
$ kubectl create -f pod-definition.yaml
```
#### Các Nhà Cung Cấp

![EKS](/images/part1/1-9/00020.png?featherlight=false&width=90pc)

#### Tài liệu Tham Khảo Lớp Lưu Trữ Kubernetes

- https://kubernetes.io/docs/concepts/storage/storage-classes/
- https://cloud.google.com/kubernetes-engine/docs/concepts/persistent-volumes#storageclasses
- https://docs.aws.amazon.com/eks/latest/userguide/storage-classes.html