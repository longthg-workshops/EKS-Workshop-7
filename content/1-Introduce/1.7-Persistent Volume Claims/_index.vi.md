---
title: "Persistent Volume Claims"
date: "`r Sys.Date()`"
weight: 7
chapter: false
pre: "<b> 1.7 </b>"
---

#### Yêu cầu Khối Lưu Trữ bền (PersistentVolumeClaim - PVC)

  - Tham khảo [Bài giảng này](https://kodekloud.com/topic/persistent-volume-claims-4/)

Trong phần này, chúng ta sẽ xem xét **PVC**

- Bây giờ chúng ta sẽ tạo một PVC để làm cho lưu trữ có sẵn cho node.
- PersistentVolume (PV) và PVC là hai đối tượng riêng biệt trong không gian tên Kubernetes.
- Khi PVC được tạo, Kubernetes sẽ gán các PV vào yêu cầu dựa trên yêu cầu và thuộc tính được thiết lập trên khối.

![PVC](/EKS-Workshop-7/images/part1/1-7/00016.png?featherlight=false&width=90pc)

- Nếu các thuộc tính không khớp hoặc Khối Lưu Trữ không có sẵn cho PVC thì nó sẽ hiển thị trạng thái `pending`.

```yaml
pvc-definition.yaml

kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes: [ "ReadWriteOnce" ]
  resources:
   requests:
     storage: 1Gi
```

```yaml
pv-definition.yaml

kind: PersistentVolume
apiVersion: v1
metadata:
    name: pv-vol1
spec:
    accessModes: [ "ReadWriteOnce" ]
    capacity:
     storage: 1Gi
    hostPath:
     path: /tmp/data
```

#### Tạo Khối Lưu Trữ

```
$ kubectl create -f pv-definition.yaml
persistentvolume/pv-vol1 is created

$ kubectl get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv-vol1   1Gi        RWO            Retain           Available                                   10s
```


#### Tạo Yêu Cầu Khối Lưu Trữ

```
$ kubectl create -f pvc-definition.yaml
persistentvolumeclaim/myclaim is created

$ kubectl get pvc
NAME      STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
myclaim   Pending                                                     35s

$ kubectl get pvc
NAME      STATUS   VOLUME    CAPACITY   ACCESS MODES   STORAGECLASS   AGE
myclaim   Bound    pv-vol1   1Gi        RWO                           1min

```

#### Xóa Yêu Cầu Khối Lưu Trữ

```
$ kubectl delete pvc myclaim
```

#### Xóa Khối Lưu Trữ

```
$ kubectl delete pv pv-vol1
```


#### Tài liệu Tham Khảo về Yêu Cầu Khối Lưu Trữ Kubernetes

- https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims
- https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#persistentvolumeclaim-v1-core
- https://docs.cloud.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengcreatingpersistentvolumeclaim.htm