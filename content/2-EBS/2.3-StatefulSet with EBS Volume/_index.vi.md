---
title: "StatefulSet với EBS Volume"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 2.3 </b>"
---

#### Cập nhật MySQL DB cho dịch vụ Catalog sử dụng EBS Volume

Bây giờ khi chúng ta đã hiểu về [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) và [Dynamic Volume Provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/), hãy thay đổi MySQL DB trên dịch vụ Catalog để cung cấp một EBS volume mới để lưu trữ các tập tin cơ sở dữ liệu một cách liên tục.

![MySQL với EBS](/images/part2/2-3/0001-mysql-ebs.png)

Sử dụng Kustomize, chúng ta sẽ thực hiện hai việc:

- Tạo một StatefulSet mới cho cơ sở dữ liệu MySQL được sử dụng bởi thành phần catalog sử dụng một EBS volume.
- Cập nhật thành phần `catalog` để sử dụng phiên bản cơ sở dữ liệu mới này.

:::info
Tại sao chúng ta không cập nhật StatefulSet hiện có? Các trường chúng ta cần cập nhật là bất biến và không thể thay đổi.
:::

Ở đây trong StatefulSet cơ sở dữ liệu catalog mới:

```file
manifests/modules/fundamentals/storage/ebs/statefulset-mysql.yaml
```

Chú ý đến trường `volumeClaimTemplates` mô tả hướng dẫn cho Kubernetes sử dụng Dynamic Volume Provisioning để tạo một EBS Volume mới, một [PersistentVolume (PV)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) và một [PersistentVolumeClaim (PVC)](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) tất cả đều tự động.

Đây là cách chúng ta sẽ cấu hình lại thành phần catalog để sử dụng StatefulSet mới:

```kustomization
modules/fundamentals/storage/ebs/deployment.yaml
Deployment/catalog
```

Áp dụng các thay đổi và chờ đợi các Pod mới được triển khai:

```bash hook=check-pvc
$ kubectl apply -k ~/environment/eks-workshop/modules/fundamentals/storage/ebs/
$ kubectl rollout status --timeout=100s statefulset/catalog-mysql-ebs -n catalog
```

Bây giờ hãy xác nhận rằng StatefulSet mới triển khai của chúng ta đang chạy:

```bash
$ kubectl get statefulset -n catalog catalog-mysql-ebs
NAME                READY   AGE
catalog-mysql-ebs   1/1     79s
```

Kiểm tra StatefulSet `catalog-mysql-ebs` của chúng ta, chúng ta có thể thấy rằng bây giờ chúng ta có một PersistentVolumeClaim được đính kèm với dung lượng 30GiB và với `storageClassName` là gp2.

```bash
$ kubectl get statefulset -n catalog catalog-mysql-ebs \
  -o jsonpath='{.spec.volumeClaimTemplates}' | jq .
[
  {
    "apiVersion": "v1",
    "kind": "PersistentVolumeClaim",
    "metadata": {
      "creationTimestamp": null,
      "name": "data"
    },
    "spec": {
      "accessModes": [
        "ReadWriteOnce"
      ],
      "resources": {
        "requests": {
          "storage": "30Gi"
        }
      },
      "storageClassName": "gp2",
      "volumeMode": "Filesystem"
    },
    "status": {
      "phase": "Pending"
    }
  }
]
```

Chúng ta có thể phân tích cách Dynamic Volume Provisioning tạo một PersistentVolume (PV) tự động cho chúng ta:

```bash
$ kubectl get pv | grep -i catalog
pvc-1df77afa-10c8-4296-aa3e-cf2aabd93365   30Gi       RWO            Delete           Bound         catalog/data-catalog-mysql-ebs-0          gp2                            10m
```

Sử dụng [AWS CLI](https://aws.amazon.com/cli/), chúng ta có thể kiểm tra volume Amazon EBS đã được tạo tự động cho chúng ta:

```bash
$ aws ec2 describe-volumes \
    --filters Name=tag:kubernetes.io/created-for/pvc/name,Values=data-catalog-mysql-ebs-0 \
    --query "Volumes[*].{ID:VolumeId,Tag:Tags}" \
    --no-cli-pager
```

Nếu bạn muốn, bạn cũng có thể kiểm tra nó qua [bảng điều khiển AWS](https://console.aws.amazon.com/ec2/home#Volumes), chỉ cần tìm kiếm các EBS volumes với tag có key `kubernetes.io/created-for/pvc/name` và giá trị là `data-catalog-mysql-ebs-0`:

![EBS Volume AWS Console Screenshot](/images/part2/2-3/0002-ebsVolumeScrenshot.png)

Nếu bạn muốn kiểm tra shell của container và kiểm tra ổ đĩa EBS mới được gắn vào hệ điều hành Linux, hãy chạy lệnh này để thực thi một lệnh shell vào container `catalog-mysql-ebs`. Nó sẽ kiểm tra các hệ thống tập tin mà bạn đã gắn kết:

```bash
$ kubectl exec --stdin catalog-mysql-ebs-0  -n catalog -- bash -c "df -h"
Filesystem      Size  Used Avail Use% Mounted on
overlay         100G  7.6G   93G   8% /
tmpfs            64M     0   64M   0% /dev
tmpfs           3.8G     0  3.8G   0% /sys/fs/cgroup
/dev/nvme0n1p1  100G  7.6G   93G   8% /etc/hosts
shm              64M     0   64M   0% /dev/shm
/dev/nvme1n1     30G  211M   30G   1% /var/lib/mysql
tmpfs           7.0G   12K  7.0G   1% /run/secrets/kubernetes.io/serviceaccount
tmpfs           3.8G     0  3.8G   0% /proc/acpi
tmpfs           3.8G     0  3.8G   0% /sys/firmware
```

Kiểm tra ổ đĩa hiện đang được gắn vào `/var/lib/mysql`. Đây là Ổ đĩa EBS cho các tập tin cơ sở dữ liệu MySQL có trạng thái được lưu trữ theo cách liên tục.

Bây giờ, hãy thử nghiệm xem dữ liệu của chúng ta có thực sự liên tục hay không. Chúng ta sẽ tạo tệp `test.txt` giống hệt như chúng ta đã làm trong phần đầu tiên của module này:

```bash
$ kubectl exec catalog-mysql-ebs-0 -n catalog -- bash -c  "echo 123 > /var/lib/mysql/test.txt"
```

Bây giờ, hãy xác minh rằng tệp `test.txt` của chúng ta đã được tạo trong thư mục `/var/lib/mysql`:

```bash
$ kubectl exec catalog-mysql-ebs-0 -n catalog -- ls -larth /var/lib/mysql/ | grep -i test
-rw-r--r-- 1 root  root     4 Oct 18 13:57 test.txt
```

Bây giờ, hãy xóa Pod hiện tại `catalog-mysql-ebs`, điều này sẽ buộc bộ điều khiển StatefulSet tự động tạo lại nó:

```bash hook=pod-delete
$ kubectl delete pods -n catalog catalog-mysql-ebs-0
pod "catalog-mysql-ebs-0" deleted
```

Đợi vài giây, và chạy lệnh dưới đây để kiểm tra xem Pod `catalog-mysql-ebs` đã được tạo lại chưa:

```bash
$ kubectl wait --for=condition=Ready pod -n catalog \
  -l app.kubernetes.io/component=mysql-ebs --timeout=60s
pod/catalog-mysql-ebs-0 condition met
$ kubectl get pods -n catalog -l app.kubernetes.io/component=mysql-ebs
NAME                  READY   STATUS    RESTARTS   AGE
catalog-mysql-ebs-0   1/1     Running   0          29s
```

Cuối cùng, hãy thực thi trở lại vào shell container MySQL và chạy lệnh `ls` trên đường dẫn `/var/lib/mysql` để tìm kiếm tệp `test.txt` mà chúng ta đã tạo, và xem liệu tệp đã được lưu liên tục hay không:

```bash
$ kubectl exec catalog-mysql-ebs-0 -n catalog -- ls -larth /var/lib/mysql/ | grep -i test
-rw-r--r-- 1 mysql root     4 Oct 18 13:57 test.txt
$ kubectl exec catalog-mysql-ebs-0 -n catalog -- cat /var/lib/mysql/test.txt
123
```

Như bạn có thể thấy, tệp `test.txt` vẫn có sẵn sau khi xóa và khởi động lại Pod và với nội dung đúng trên đó là `123`. Đây là chức năng chính của Khối lưu trữ liên tục (PVs). Amazon EBS đang lưu trữ dữ liệu và giữ dữ liệu của chúng ta an toàn và có sẵn trong một khu vực khả dụng của AWS.