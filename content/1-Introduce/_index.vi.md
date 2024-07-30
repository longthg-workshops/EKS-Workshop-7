---
title: "Giới thiệu"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 1. </b>"
---

#### Hiểu về lưu trữ trong Kubernetes

Trong chương này, chúng ta sẽ tập trung tìm hiểu một số vấn đề về lưu trữ trong Kubernetes, bao gồm PersistentVolume (PV), PersistentVolumeClaim (PVC) và StorageClass. Đồng thời, chúng ta cũng tìm hiểu về một số hệ thống lưu trữ trên AWS dành cho EKS, bao gồm EBS và EFS.

- **Các phần cần tìm hiểu:**
    - Lưu trữ đối với container
      - Lưu trữ trên docker
      - Plugin cho trình điều kiển lưu trữ Docker
      - CSI
    - Volume trên kubernetes
      - Volume
      - PersistentVolume
      - PersistentVolumeClaim
    - Lớp lưu trữ (StorageClass)
 