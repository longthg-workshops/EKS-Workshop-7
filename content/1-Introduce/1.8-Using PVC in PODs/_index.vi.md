---
title: "Sử dụng PVC trong các pod"
date: "`r Sys.Date()`"
weight: 8
chapter: false
pre: "<b> 1.8 </b>"
---

#### Sử dụng PVC trong PODs

  - Đi đến [Bài giảng](https://kodekloud.com/topic/using-pvc-in-pods/)

Trong phần này, chúng ta sẽ tìm hiểu về **Sử dụng PVC trong PODs**

- Các pod truy cập vào bộ nhớ lưu trữ bằng cách khẳng định vùng lưu trữ (volume). PersistentVolumeClaim phải tồn tại trong cùng namespace với Pod yêu cầu.
- Cụm sẽ tìm thấy yêu cầu trong namespace của Pod và sử dụng nó để lấy PersistentVolume laim. Vùng lưu trữ sau đó được gắn vào máy chủ và vào Pod.
- PersistentVolume tồn tại trong phạm vi cụm và PersistentVolumeClaim tồn tại trong phạm vi namespace.

#### Tạo Persistent Volume

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
```
$ kubectl create -f pv-definition.yaml

```

#### Tạo Persistent Volume Claim

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
```
$ kubectl create -f pvc-definition.yaml
```

#### Tạo Pod

```yaml
pod-definition.yaml

apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
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

#### Liệt kê các Pod, Persistent Volume và Persistent Volume Claim

```
$ kubectl get pod,pvc,pv

```

#### Tài liệu tham khảo

- https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes