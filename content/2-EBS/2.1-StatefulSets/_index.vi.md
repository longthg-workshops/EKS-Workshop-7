---
title: "StatefulSets"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 2.1 </b>"
---

#### StatefulSets

Tương tự như Deployments, [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/) quản lý các Pods dựa trên một container spec đồng nhất. Khác với Deployments, StatefulSets duy trì một danh tính cố định cho mỗi Pod của nó. Các Pods này được tạo ra từ cùng một spec, nhưng không thể hoán đổi với nhau, mỗi Pod có một định danh bền vững mà nó duy trì qua bất kỳ sự kiện lên lịch lại nào.

Nếu bạn muốn sử dụng các container lưu trữ để cung cấp tính bền vững cho công việc của mình, bạn có thể sử dụng StatefulSet như một phần của giải pháp. Mặc dù các Pods riêng lẻ trong một StatefulSet có thể gặp sự cố, các định danh Pod bền vững giúp các Pod mới thay thế Pod lỗi dễ dàng thích nghi với các pod đang hoạt động hơn.

StatefulSets có ý nghĩa với các ứng dụng yêu cầu một hoặc nhiều trong các yếu tố sau:

- Các định danh mạng ổn định, duy nhất
- Lưu trữ ổn định, bền vững
- Triển khai và mở rộng dịch vụ một cách có tổ chức và an toàn
- Cập nhật tự động và có tổ chức

Trong ứng dụng thương mại điện tử của workshop, chúng ta đã triển khai một StatefulSet như một phần của vi dịch vụ Catalog. Vi dịch vụ Catalog sử dụng cơ sở dữ liệu MySQL chạy trên EKS. Cơ sở dữ liệu là một ví dụ tốt cho việc sử dụng StatefulSets vì chúng đòi hỏi **lưu trữ bền vững**. Chúng ta có thể phân tích Pod MySQL Database của chúng ta để xem cấu hình container hiện tại của nó:

```bash
$ kubectl describe statefulset -n catalog catalog-mysql
Name:               catalog-mysql
Namespace:          catalog
[...]
  Containers:
   mysql:
    Image:      public.ecr.aws/docker/library/mysql:5.7
    Port:       3306/TCP
    Host Port:  0/TCP
    Args:
      --ignore-db-dir=lost+found
    Environment:
      MYSQL_ROOT_PASSWORD:  my-secret-pw
      MYSQL_USER:           <set to the key 'username' in secret 'catalog-db'>  Optional: false
      MYSQL_PASSWORD:       <set to the key 'password' in secret 'catalog-db'>  Optional: false
      MYSQL_DATABASE:       <set to the key 'name' in secret 'catalog-db'>      Optional: false
    Mounts:
      /var/lib/mysql from data (rw)
  Volumes:
   data:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
Volume Claims:  <none>
[...]
```

Như bạn có thể thấy phần [`Volumes`](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir-configuration-example) của StatefulSet của chúng ta cho thấy chúng ta chỉ sử dụng một loại container [EmptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir) mà "chia sẻ với tuổi thọ của Pod".

![MySQL với emptyDir](/EKS-Workshop-7/images/part2/2-1/mysql-emptydir.png)

Một container `emptyDir` được tạo ra khi một Pod được gán cho một nút, và tồn tại trong suốt thời gian Pod đó đang chạy trên một nút. Như tên gọi của nó, container emptyDir ban đầu rỗng. Tất cả các container trong Pod có thể đọc và ghi các tệp tin trong emptyDir, mặc dù container đó có thể được gắn vào các đường dẫn giống nhau hoặc khác nhau trong mỗi container. **Khi một Pod được xóa khỏi một nút vì bất kỳ lý do nào, dữ liệu trong emptyDir sẽ bị xóa vĩnh viễn.** Do đó, EmptyDir không phù hợp cho cơ sở dữ liệu MySQL của chúng ta.

Chúng ta có thể minh họa điều này bằng cách bắt đầu một phiên shell bên trong container MySQL và tạo một tệp tin thử nghiệm. Sau đó, chúng ta sẽ xóa Pod đang chạy trong StatefulSet của chúng ta. Bởi vì Pod đang sử dụng emptyDir và không phải là Persistent Volume (PV), tệp tin sẽ không tồn tại sau khi Pod được khởi động lại. Đầu tiên, hãy chạy một lệnh bên trong container MySQL của chúng ta để tạo một tệp tin trong đường dẫn emptyDir `/var/lib/mysql` (nơi MySQL lưu trữ các tệp cơ sở dữ liệu):

```bash
$ kubectl exec catalog-mysql-0 -n catalog -- bash -c  "echo 123 > /var/lib/mysql/test.txt"
```

Bây giờ, hãy xác minh rằng tệp `test.txt` của chúng ta đã được tạo trong thư mục `/var/lib/mysql`:

```bash
$ kubectl exec catalog-mysql-0 -n catalog -- ls -larth /var/lib/mysql/ | grep -i test
-rw-r--r-- 1 root  root     4 Oct 18 13:38 test.txt
```

Bây giờ, hãy xóa Pod `catalog-mysql` hiện tại. Điều này sẽ buộc bộ điều khiển StatefulSet tự động tạo lại một Pod `catalog-mysql` mới:

```bash
$ kubectl delete pods -n catalog -l app.kubernetes.io/component=mysql
pod "catalog-mysql-0" deleted
```

Chờ một vài giây và chạy lệnh dưới đây để kiểm tra xem Pod `catalog-mysql` đã được tạo lại chưa:

```bash
$ kubectl wait --for=condition=Ready pod -n catalog \
  -l app.kubernetes.io/component=mysql --timeout=30s
pod/catalog-mysql-0 condition met
$ kubectl get pods -n catalog -l app.kubernetes.io/component=mysql
NAME              READY   STATUS    RESTARTS   AGE
catalog-mysql-0   1/1     Running   0          29s
```

Cuối cùng, hãy thực hiện lại vào shell của container MySQL và chạy một lệnh `ls` trong đường dẫn `/var/lib/mysql` để tìm tệp `test.txt` đã được tạo trước đó:

```bash expectError=true
$ kubectl exec catalog-mysql-0 -n catalog -- cat /var/lib/mysql/test.txt
cat: /var/lib/mysql/test.txt: No such file or directory
command terminated with exit code 1
```

Như bạn có thể thấy, tệp `test.txt` không còn tồn tại nữa do các thư mục `emptyDir` là tạm thời. Trong các phần tiếp theo, chúng ta sẽ thực hiện cùng một thử nghiệm và thể hiện cách Persistent Volumes (PVs) sẽ duy trì tệp `test.txt` và tồn tại qua các lần khởi động lại và/hoặc sự cố của Pod.

Trên trang tiếp theo, chúng ta sẽ làm việc để hiểu các khái niệm chính về Lưu trữ trên Kubernetes và tích hợp của nó với hệ sinh thái đám mây AWS.