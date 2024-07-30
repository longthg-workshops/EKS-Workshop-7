---
title: "Triển khai"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 4.1 </b>"
---

Trên ứng dụng thương mại điện tử trong bài lab này, chúng ta đã tạo một bản triển khai như một phần của vi dịch vụ `assets`. Vi dịch vụ `assets` sử dụng máy chủ web chạy trên EKS. Máy chủ web là một ví dụ tuyệt vời cho việc triển khai Kubernetes vì chúng _mở rộng theo chiều ngang_ và **khai báo trạng thái mới** của Pod.

Một thành phần `assets` là một container phân phối các tệp ảnh tĩnh cho các sản phẩm, các ảnh minh họa sản phẩm được thêm vào như một phần của bản dựng image container. Tuy nhiên với cách bố trí này, mỗi lần đội ngũ muốn cập nhật hình ảnh, họ phải dựng và triển khai lại image của container. Trong bài tập này, chúng ta sẽ sử dụng [Amazon FSx for NetApp ONTAP File System](https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/what-is-fsx-ontap.html) và Kubernetes [Persistent Volume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) để cập nhật các ảnh minh họa sản phẩm cũ và thêm các ảnh mới mà không cần phải dựng lại image cho container từ đầu.

Ta có thể bắt đầu bằng việc xem mô tả bản triển khai để biết cấu hình volume ban đầu:

```bash
$ kubectl describe deployment -n assets
Name:                   assets
Namespace:              assets
[...]
  Containers:
   assets:
    Image:      public.ecr.aws/aws-containers/retail-store-sample-assets:latest
    Port:       8080/TCP
    Host Port:  0/TCP
    Limits:
      memory:  128Mi
    Requests:
      cpu:     128m
      memory:  128Mi
    Liveness:  http-get http://:8080/health.html delay=30s timeout=1s period=3s #success=1 #failure=3
    Environment Variables from:
      assets      ConfigMap  Optional: false
    Environment:  <none>
    Mounts:
      /tmp from tmp-volume (rw)
  Volumes:
   tmp-volume:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     Memory
    SizeLimit:  <unset>
[...]
```

Như các bạn thấy, phần [`Volume`](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir-configuration-example) của StatefulSet cho thấy chúng ta đang sử dụng loại volume [EmptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir) có cùng tuổi thọ với pod.

![Assets cùng volume emptyDir](./assets/assets-emptydir.webp)

Một volume `emptyDir` ban đầu được tạo ra khi Pod được gắn vào một node và tồn tại trong suốt thời gian pod đó vẫn chạy trên cùng một node. Như tên gọi, emptyDir ban đầu là một thư mục rỗng. Mọi container trong cùng pod có thể đọc và ghi các file lên cùng một volume `emptyDir`, mặc dù volume này có thể được gắn vào các đường dẫn khác nhau trong hệ tập tin của các container. **Khi pod bị xóa khỏi node, bất luận lý do, dữ liệu trong emptyDir cũng bị xóa vĩnh viễn.** Điều này đồng nghĩa với việc emptyDir không phải lựa chọn thích hợp nếu muốn chia sẻ dữ liệu giữa nhiều pod.

Container của chúng ta chứa các hình ảnh ban đầu của sản phẩm được chép vào như một phần của bản dựng ban đầu, đặt trong thư mục `/usr/share/nginx/html/assets`. Chúng ta có thể kiểm tra với lệnh sau:

```bash
$ kubectl exec --stdin deployment/assets \
  -n assets -- bash -c "ls /usr/share/nginx/html/assets/"
chrono_classic.jpg
gentleman.jpg
pocket_watch.jpg
smart_1.jpg
smart_2.jpg
wood_watch.jpg
```

Trước hết, chúng ta mở rộng quy mô của bản triển khai `deployment/assets`:

```bash
$ kubectl scale -n assets --replicas=2 deployment/assets
$ kubectl rollout status -n assets deployment/assets --timeout=60s
```

Giờ hãy thử đặt ảnh sản phẩm `newproduct.png` vào thư mục `/usr/share/nginx/html/assets` của pod đầu tiên bằng lệnh sau:

```bash
$ POD_NAME=$(kubectl -n assets get pods -o jsonpath='{.items[0].metadata.name}')
$ kubectl exec $POD_NAME \
  -n assets -- bash -c 'touch /usr/share/nginx/html/assets/newproduct.png'
```

Bước tiếp theo, ta xác nhận ảnh `newproduct.png` vừa tạo chưa xuất hiện trên pod thứ hai:

```bash
$ POD_NAME=$(kubectl -n assets get pods -o jsonpath='{.items[1].metadata.name}')
$ kubectl exec $POD_NAME \
  -n assets -- bash -c 'ls /usr/share/nginx/html/assets'
```

Như các bạn thấy, ảnh `newproduct.png` chưa xuất hiện trên pod thứ hai. Để giải quyết vấn đề này, ta cần một hệ tập tin chung ho các pod nếu dịch vụ cần mở rộng theo chiều ngang trong khi ta vẫn có thể cập nhật các tập tin mà không cần tái triển khai.

![Assets with FSxN](./assets/assets-fsxn.webp)
