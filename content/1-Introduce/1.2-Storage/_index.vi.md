---
title: "Hệ thống tệp trong Docker"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 1.2 </b>"
---

#### Lưu trữ trong Docker

Trong phần này, chúng ta sẽ xem xét **Driver Lưu trữ và Hệ thống tệp của Docker**.

- Chúng ta sẽ xem xét nơi và cách Docker lưu trữ dữ liệu và cách quản lý hệ thống tệp của các container.

#### Hệ thống tệp

- Hãy xem cách Docker lưu trữ dữ liệu trên hệ thống tệp cục bộ. Khi bạn cài đặt Docker lần đầu tiên trên một hệ thống, nó tạo ra cấu trúc thư mục này tại /var/lib/docker.

  ```
  $ cd /var/lib/docker/

  ```

![EKS](/images/part1/1-2/0002.png?featherlight=false&width=90pc)

- Bạn có nhiều thư mục con dưới đó gọi là `aufs, containers, image, volumes, vv.`
- Đây là nơi mà Docker lưu trữ tất cả dữ liệu của theo mặc định.
- Tất cả các tệp liên quan đến các container được lưu trữ dưới thư mục `containers` và các tệp liên quan đến các image được lưu trữ dưới thư mục `image`. Bất kỳ thư mục nào được tạo bởi các container Docker đều được tạo dưới thư mục `volumes`.
- Bây giờ, chỉ cần hiểu nơi Docker lưu trữ các tệp image và container cũng như các định dạng.
- Để hiểu điều đó, chúng ta cần hiểu về kiến ​​trúc phân lớp của Docker.

#### Kiến ​​trúc lớp

- Khi Docker xây dựng image, nó xây dựng chúng trong theo kiến ​​trúc lớp. Mỗi dòng lệnh trong tệp Docker tạo một lớp mới trong Docker image chỉ từ các thay đổi từ lớp trước đó.
- Như bạn có thể thấy trong ảnh, lớp-1 là lớp cơ sở Ubuntu, lớp-2 cài đặt các gói, lớp-3 cài đặt các gói Python, lớp-4 cập nhật mã nguồn, lớp-5 cập nhật điểm vào của image. Khi mỗi lớp chỉ lưu trữ các thay đổi từ lớp trước đó. Nó cũng được phản ánh trong kích thước.

![EKS](/images/part1/1-2/0003.png?featherlight=false&width=90pc)

- Để hiểu rõ hơn về các ưu điểm của kiến ​​trúc lớp này, hãy xem một Dockerfile khác. Dockerfile này rất giống với Dockerfile đầu tiên, chỉ khác nhau ở mã nguồn ứng dụng và entrypoint. 
- Khi image được xây dựng, Docker sẽ không xây dựng ba lớp đầu tiên mà thay vào đó sẽ sử dụng lại ba lớp giống như các lớp đã xây dựng cho ứng dụng đầu tiên từ bộ nhớ cache. Chỉ tạo ra hai lớp cuối cùng với nguồn và điểm vào mới. 
- Như vậy, Docker xây dựng image nhanh chóng và hiệu quả tiết kiệm không gian đĩa. Điều này cũng áp dụng nếu bạn muốn cập nhật mã ứng dụng của mình. Docker đơn giản là sử dụng lại tất cả các lớp trước đó từ bộ nhớ cache và nhanh chóng xây dựng lại image ứng dụng bằng cách cập nhật mã nguồn mới nhất. 

![EKS](/images/part1/1-2/0004.png?featherlight=false&width=90pc)

- Hãy sắp xếp lại các lớp từ dưới lên để chúng ta có thể hiểu rõ hơn. Tất cả các lớp này được tạo ra khi chúng ta chạy lệnh Docker build để hình thành Docker image cuối cùng. Khi quá trình xây dựng hoàn tất, bạn không thể chỉnh sửa nội dung của các lớp này và vì vậy chúng là chỉ đọc và bạn chỉ có thể chỉnh sửa chúng bằng cách khởi tạo một quá trình xây dựng mới.

![EKS](/images/part1/1-2/0005.png?featherlight=false&width=90pc)

- Khi bạn chạy một container dựa trên image này, bằng cách sử dụng lệnh **Docker run** Docker tạo ra một container dựa trên các lớp này và tạo ra một lớp có thể ghi mới trên lớp image. Lớp có thể ghi được sử dụng để lưu trữ dữ liệu cũng như các tệp tạm thời được tạo ra bởi container (VD: các tệp log được xuất từ các ứng dụng).

![EKS](/images/part1/1-2/0006.png?featherlight=false&width=90pc)

- Khi container bị hủy, lớp này và tất cả các thay đổi được lưu trữ trong đó cũng bị hủy. Hãy nhớ rằng tất cả các container chia sẻ cùng một lớp image chung (và không thay đổi trong toàn bộ vòng đời của các container), song vẫn độc lập với nhau trong môi trường hoạt động.