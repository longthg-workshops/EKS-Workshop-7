---
title: "Cấp phát động sử dụng EFS"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 3.3 </b>"
---

#### Cấp phát động sử dụng EFS

Bây giờ khi chúng ta đã hiểu về lớp lưu trữ EFS cho Kubernetes, hãy tạo một [Persistent Volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) và thay đổi container `assets` trên deployment assets để mount Volume được tạo.

Đầu tiên, hãy kiểm tra tệp `efspvclaim.yaml` để xem các tham số trong tệp và yêu cầu về kích thước lưu trữ cụ thể là 5GB từ lớp lưu trữ `efs-sc` mà chúng ta đã tạo trong bước trước:

```file
manifests/modules/fundamentals/storage/efs/deployment/efspvclaim.yaml
```

Chúng ta cũng sẽ sửa đổi dịch vụ assets theo hai cách:

- Mount PVC vào vị trí nơi các hình ảnh assets được lưu trữ
- Thêm một [init container](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/) để sao chép các hình ảnh ban đầu vào thư mục EFS

```kustomization
modules/fundamentals/storage/efs/deployment/deployment.yaml
Deployment/assets
```

Chúng ta có thể áp dụng các thay đổi bằng cách chạy lệnh sau:

```bash
$ kubectl apply -k ~/environment/eks-workshop/modules/fundamentals/storage/efs/deployment
namespace/assets không thay đổi
serviceaccount/assets không thay đổi
configmap/assets không thay đổi
service/assets không thay đổi
persistentvolumeclaim/efs-claim được tạo
deployment.apps/assets được cấu hình
$ kubectl rollout status --timeout=130s deployment/assets -n assets
```

Bây giờ hãy nhìn vào `volumeMounts` trong deployment, chú ý rằng chúng ta có `Volume` mới của mình được đặt tên là `efsvolume` được mount trên `volumeMounts` với tên `/usr/share/nginx/html/assets`:

```bash
$ kubectl get deployment -n assets \
  -o yaml | yq '.items[].spec.template.spec.containers[].volumeMounts'
- mountPath: /usr/share/nginx/html/assets
  name: efsvolume
- mountPath: /tmp
  name: tmp-volume
```

#### Tạo Một PersistentVolume (PV) Tự Động cho PersistentVolumeClaim (PVC)

Trong bước trước đó, chúng ta đã tạo một PersistentVolumeClaim (PVC), và một PersistentVolume (PV) đã được tạo tự động cho nó:

```bash
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                 STORAGECLASS   REASON   AGE
pvc-342a674d-b426-4214-b8b6-7847975ae121   5Gi        RWX            Delete           Bound    assets/efs-claim                      efs-sc                  2m33s
```

Mô tả thêm về PersistentVolumeClaim (PVC) được tạo:

```bash
$ kubectl describe pvc -n assets
Name:          efs-claim
Namespace:     assets
StorageClass:  efs-sc
Status:        Bound
Volume:        pvc-342a674d-b426-4214-b8b6-7847975ae121
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: efs.csi.aws.com
               volume.kubernetes.io/storage-provisioner: efs.csi.aws.com
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      5Gi
Access Modes:  RWX
VolumeMode:    Filesystem
Used By:       <none>
Events:
  Type    Reason                 Age   From                                                                                      Message
  ----    ------                 ----  ----                                                                                      -------
  Normal  ExternalProvisioning   34s   persistentvolume-controller                                                               waiting for a volume to be created, either by external provisioner "efs.csi.aws.com" or manually created by system administrator
  Normal  Provisioning           34s   efs.csi.aws.com_efs-csi-controller-6b4ff45b65-fzqjb_7efe91cc-099a-45c7-8419-6f4b0a4f9e01  External provisioner is provisioning volume for claim "assets/efs-claim"
  Normal  ProvisioningSucceeded  33s   efs.csi.aws.com_efs-csi-controller-6b4ff45b65-fzqjb_7efe91cc-099a-45c7-8419-6f4b0a4f9e01  Successfully provisioned volume pvc-342a674d-b426-4214-b8b6-7847975ae121
```

#### Tạo Một File Mới `newproduct.png` Trong Thư Mục Assets của Pod Đầu Tiên

```bash
$ POD_NAME=$(kubectl -n assets get pods -o jsonpath='{.items[0].metadata.name}')
$ kubectl exec --stdin $POD_NAME \
  -n assets -c assets -- bash -c 'touch /usr/share/nginx/html/assets/newproduct.png'
```

Sau đó, xác nhận rằng tập tin cũng tồn tại trong Pod thứ hai:

```bash
$ POD_NAME=$(kubectl -n assets get pods -o jsonpath='{.items[1].metadata.name}')
$ kubectl exec --stdin $POD_NAME \
  -n assets -c assets -- bash -c 'ls /usr/share/nginx/html/assets'
chrono_classic.jpg
gentleman.jpg
newproduct.png <-----------
pocket_watch.jpg
smart_1.jpg
smart_2.jpg
test.txt
wood_watch.jpg
```

Như bạn có thể thấy, ngay cả khi chúng ta tạo một tập tin thông qua Pod đầu tiên, Pod thứ hai cũng có quyền truy cập vào tập tin này nhờ vào hệ thống tệp EFS được chia sẻ.