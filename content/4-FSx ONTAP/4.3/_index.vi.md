---
title: "Cấp phát động bằng FSx for NetApp ONTAP"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 4.3 </b>"
---

Giờ đây chúng ta đã hiểu về lớp lưu trữ FSxN cho Kubernetes, hãy tạo một [PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) và sửa container `assets` trong bản triển khai cùng tên để gắn volume vừa tạo.


Trước hết hãy đi sâu vào tệp `fsxnpvclaim.yaml` để xem các tham số trong file cùng phần khai báo kích thước cụ thể (5GB) trong lớp lưu trữ `fsxn-sc-nfs`.

```file
manifests/modules/fundamentals/storage/fsxn/deployment/fsxnpvclaim.yaml
```
```kustomization
modules/fundamentals/storage/fsxn/deployment/deployment.yaml
Deployment/assets
```

Ta có thể áp dụng các thay đổi bằng lệnh sau:
```bash
$ kubectl apply -k ~/environment/eks-workshop/modules/fundamentals/storage/fsxn/deployment
namespace/assets unchanged
serviceaccount/assets unchanged
configmap/assets unchanged
service/assets unchanged
persistentvolumeclaim/fsxn-nfs-claim created
deployment.apps/assets configured

$ kubectl rollout status --timeout=130s deployment/assets -n assets
```
Giờ hãy nhìn vào bản triển khai `volumeMounts`, hãy để ý rằng chúng ta đã có volume mới với tên `efsvolume` được gắn vào `volumeMounts` với tên gọi và đường dẫn `/usr/share/nginx/html/assets`:

```bash
$ kubectl get deployment -n assets \
  -o yaml | yq '.items[].spec.template.spec.containers[].volumeMounts'
- mountPath: /usr/share/nginx/html/assets
  name: fsxnvolume
- mountPath: /tmp
  name: tmp-volume
```

Một PersistentVolume (PV) được tạo tự động để dành cho PersistentVolumeClaim (PVC) chúng ta đã tạo trước đó.

Dùng lệnh "describe" để mô tả PVC vừa tạo.

```bash
$ kubectl describe pvc -n assets
Name:          fsxn-nfs-claim
Namespace:     assets
StorageClass:  fsxn-sc-nfs
Status:        Bound
Volume:        pvc-ceec6f39-8034-4b33-a4bc-c1b1370befd1
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: csi.trident.netapp.io
               volume.kubernetes.io/storage-provisioner: csi.trident.netapp.io
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      5Gi
Access Modes:  RWX
VolumeMode:    Filesystem
Used By:       assets-555dc4c9c9-g8hfs
               assets-555dc4c9c9-m6r2l
Events:        <none>
```

Giờ tạo một tệp mới với tên `newproduct.png` trong thư mục "assets" trong Pod thứ nhất:

```bash
$ POD_NAME=$(kubectl -n assets get pods -o jsonpath='{.items[0].metadata.name}')
$ kubectl exec --stdin deployment/assets $POD_NAME \
  -n assets -- bash -c 'touch /usr/share/nginx/html/assets/newproduct.png'
```

Chạy lệnh xác nhận file đó cũng ở trong Pod thứ hai:

```bash
$ POD_NAME=$(kubectl -n assets get pods -o jsonpath='{.items[1].metadata.name}')
$ kubectl exec --stdin deployment/assets $POD_NAME \
  -n assets -- bash -c 'ls /usr/share/nginx/html/assets'
chrono_classic.jpg
gentleman.jpg
newproduct.png <-----------
pocket_watch.jpg
smart_1.jpg
smart_2.jpg
test.txt
wood_watch.jpg
```
Giờ bạn thấy tuy chúng ta tạo file cho pod thứ nhất, pod thứ hai vẫn truy cập được vì cả 2 chia chung hệ tập tin FSxN.
