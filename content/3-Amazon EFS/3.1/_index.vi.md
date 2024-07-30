---
title: "Lưu trữ lâu dài qua mạng"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 3.1 </b>"
---

#### Lưu trữ lâu dài qua mạng

Trên ứng dụng thương mại điện tử của chúng tôi, chúng tôi đã tạo một `deployment` như là một phần của dịch vụ microservice của chúng tôi. Dịch vụ microservice sử dụng một máy chủ web chạy trên EKS. Các máy chủ web là một ví dụ tuyệt vời cho việc sử dụng `deployments` vì chúng **mở rộng theo chiều ngang** và **khai báo trạng thái mới** của các `Pods`.

Thành phần tài sản là một container phục vụ các hình ảnh tĩnh cho sản phẩm, các hình ảnh sản phẩm này được thêm vào như một phần của quá trình xây dựng container image. Tuy nhiên, với cài đặt này, mỗi khi nhóm muốn cập nhật hình ảnh sản phẩm, họ phải tạo và triển khai lại container image. Trong bài tập này, chúng tôi sẽ sử dụng [Hệ thống Tệp EFS](https://docs.aws.amazon.com/efs/latest/ug/whatisefs.html) và [Khối Lưu trữ Vĩnh Viễn Kubernetes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) để cập nhật hình ảnh sản phẩm cũ và thêm hình ảnh sản phẩm mới mà không cần phải xây dựng lại các container image.

Chúng ta có thể bắt đầu bằng cách mô tả `Deployment` để xem cấu hình khối lượng ban đầu của nó:

```bash
$ kubectl describe deployment -n assets
Name:                   assets
Namespace:              assets
[...]
  Containers:
   assets:
    Image:      public.ecr.aws/aws-containers/retail-store-sample-assets:0.4.0
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
    Type:       EmptyDir (một thư mục tạm thời chia sẻ với tuổi thọ của một Pod)
    Medium:     Memory
    SizeLimit:  <unset>
[...]
```

Như bạn có thể thấy phần [`Volumes`](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir-configuration-example) của `StatefulSet` của chúng ta chỉ sử dụng loại khối lượng [EmptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir) mà "chia sẻ tuổi thọ của Pod".

![Thuộc tính emptyDir](images/part3/3.1/0001-assets-emptydir.png)

Một khối lượng `emptyDir` được tạo ra khi một Pod được gán cho một nút và tồn tại miễn là Pod đó đang chạy trên nút đó. Như tên gọi, khối lượng emptyDir ban đầu là trống. Tất cả các container trong Pod có thể đọc và ghi các tập tin giống nhau trong khối lượng emptyDir, mặc dù khối lượng đó có thể được gắn vào các đường dẫn giống hoặc khác nhau trong mỗi container. **Khi một Pod được loại bỏ khỏi một nút vì bất kỳ lý do nào, dữ liệu trong emptyDir sẽ bị xóa vĩnh viễn.** Điều này có nghĩa là nếu chúng ta muốn chia sẻ dữ liệu giữa nhiều Pods trong cùng một `Deployment` và thực hiện các thay đổi đối với dữ liệu đó, thì EmptyDir không phải là một lựa chọn phù hợp.

Container có một số hình ảnh sản phẩm ban đầu được sao chép vào nó như là một phần của quá trình xây dựng container image dưới thư mục `/usr/share/nginx/html/assets`, chúng ta có thể kiểm tra bằng cách chạy lệnh sau:

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

Mở rộng `assets` Deployment để có nhiều bản sao:

```bash
$ kubectl scale -n assets --replicas=2 deployment/assets
$ kubectl rollout status -n assets deployment/assets --timeout=60s
```

Bây giờ hãy thử đặt một hình ảnh sản phẩm mới có tên là `newproduct.png` trong thư mục `/usr/share/nginx/html/assets` của Pod đầu tiên bằng lệnh sau:

```bash
$ POD_NAME=$(kubectl -n assets get pods -o jsonpath='{.items[0].metadata.name}')
$ kubectl exec --stdin $POD_NAME \
  -n assets -- bash -c 'touch /usr/share/nginx/html/assets/newproduct.png'
```

Tiếp theo, xác nhận rằng hình ảnh sản phẩm mới `newproduct.png` không tồn tại trên hệ thống tập tin của Pod thứ hai:

```bash
$ POD_NAME=$(kubectl -n assets get pods -o jsonpath='{.items[1].metadata.name}')
$ kubectl exec --stdin $POD_NAME \
  -n assets -- bash -c 'ls /usr/share/nginx/html/assets'
```

Như bạn thấy, hình ảnh mới được tạo `newproduct.png` không tồn tại trên Pod thứ hai. Để giải quyết vấn đề này, chúng ta cần một hệ thống tệp có thể được chia sẻ qua nhiều Pod nếu dịch vụ cần mở rộng theo chiều ngang trong khi vẫn thực hiện các cập nhật vào các tệp mà không cần triển khai lại.

![Assets với EFS](images/part3/3.1/0002-assets-efs.png)