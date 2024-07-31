---
title: "Persistent Volumes"
date: "`r Sys.Date()`"
weight: 6
chapter: false
pre: "<b> 1.6 </b>"
---

#### Khối Lưu Trữ Bền (Persistent Volume)

  - Điều hướng đến [Bài Giảng](https://kodekloud.com/topic/persistent-volumes-4/)

Trong phần này, chúng ta sẽ xem xét về **Khối Lưu Trữ Cố Định**

- Trong môi trường lớn, với nhiều người dùng triển khai nhiều pods, người dùng sẽ phải cấu hình lưu trữ riêng lẻ cho từng Pod.
- Bất kể giải pháp lưu trữ nào được sử dụng, người dùng triển khai các pods sẽ phải cấu hình chúng trên tất cả các tệp định nghĩa pods trong môi trường của mình. Mỗi khi có thay đổi, người dùng sẽ phải thực hiện chúng trên tất cả các pods của mình.

![EKS](/EKS-Workshop-7/images/part1/1-6/00016.png?featherlight=false&width=90pc)

- Một Khối Lưu Trữ Bền là một cụm lưu trữ trên toàn cụm được cấu hình bởi một quản trị viên để được sử dụng bởi người dùng triển khai ứng dụng trên cụm. Người dùng có thể lựa chọn lưu trữ từ cụm này bằng cách sử dụng các Yêu Cầu Khối Lưu Trữ Cố Định.

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

  ```bash
  $ kubectl create -f pv-definition.yaml
  persistentvolume/pv-vol1 created

  $ kubectl get pv
  NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
  pv-vol1   1Gi        RWO            Retain           Available                                   3min
  
  $ kubectl delete pv pv-vol1
  persistentvolume "pv-vol1" deleted
  ```

#### Khối Lưu Trữ Bền Kubernetes

- https://kubernetes.io/docs/concepts/storage/persistent-volumes/
- https://portworx.com/tutorial-kubernetes-persistent-volumes/