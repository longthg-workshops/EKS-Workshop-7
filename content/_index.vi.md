---
title: "Kubernetes trên AWS"
date: "`r Sys.Date()`"
weight: 1
chapter: false
---

# Kubernetes trên AWS

Kubernetes là một nền tảng mã nguồn mở, linh hoạt, có khả năng mở rộng, phục vụ việc quản lý các ứng dụng được đóng gói và các dịch vụ liên quan, giúp việc cấu hình và tự động hóa quá trình triển khai ứng dụng trở nên thuận tiện hơn. Được biết đến như một hệ sinh thái lớn và phát triển nhanh chóng, Kubernetes cung cấp sự hỗ trợ rộng rãi qua các dịch vụ và công cụ đa dạng.

Tên Kubernetes bắt nguồn từ tiếng Hy Lạp, nghĩa là người lái tàu hoặc hoa tiêu. Kubernetes được Google công bố mã nguồn vào năm 2014, dựa trên gần một thập kỷ kinh nghiệm quản lý workload lớn trong thực tế của Google, kết hợp với các ý tưởng và best practices từ cộng đồng.

#### Quay ngược thời gian

Hãy xem xét tại sao Kubernetes lại quan trọng thông qua việc nhìn lại quá khứ.

![Kubernetes](/EKS-Workshop-7/images/4/00010.svg?featherlight=false&width=60pc)

**Thời kỳ triển khai truyền thống:** Ban đầu, các ứng dụng được chạy trực tiếp trên máy chủ vật lý, khiến việc phân bổ tài nguyên gặp khó khăn do không có cơ chế xác định ranh giới tài nguyên cho từng ứng dụng. Cách tiếp cận này dẫn đến nguy cơ một ứng dụng có thể sử dụng quá nhiều tài nguyên, ảnh hưởng đến hoạt động của các ứng dụng khác. Giải pháp là chạy mỗi ứng dụng trên một máy chủ vật lý riêng biệt, nhưng điều này lại không hiệu quả về mặt chi phí và tài nguyên.

**Thời kỳ triển khai ảo hóa:** Ảo hóa được giới thiệu như một giải pháp cho phép chạy nhiều Máy ảo (VM) trên cùng một máy chủ vật lý, giúp cô lập ứng dụng và tăng cường bảo mật. Ảo hóa cũng giúp cải thiện hiệu quả sử dụng tài nguyên và khả năng mở rộng.

**Thời kỳ triển khai Container:** Container giống như VM nhưng nhẹ hơn và chia sẻ Hệ điều hành (HĐH) với nhau. Container mang lại nhiều lợi ích như tạo mới và triển khai ứng dụng nhanh chóng, phát triển và triển khai liên tục, phân biệt rõ ràng giữa quá trình phát triển và vận hành, cung cấp tính nhất quán qua các môi trường, khả năng di chuyển giữa các cloud và HĐH, và quản lý ứng dụng tập trung.

#### Tại sao bạn cần Kubernetes và nó có thể làm gì?

Container là phương tiện hiệu quả để đóng gói và chạy ứng dụng của bạn. Trong môi trường sản xuất, cần có cơ chế quản lý các container một cách hiệu quả, đảm bảo không có downtime. Kubernetes giúp quản lý các hệ thống phân tán mạnh mẽ, tự động hóa việc nhân rộng, cung cấp các mẫu triển khai và nhiều hơn nữa.

Kubernetes mang lại:

- **Phát hiện dịch vụ và cân bằng tải**
- **Điều phối bộ nhớ**
- **Tự động rollouts và rollbacks**
- **Đóng gói tự động**
- **Tự phục hồi**
- **Quản lý cấu hình và bảo mật**

#### Những gì Kubernetes không phải là

Kubernetes không phải là một hệ thống PaaS truyền thống, toàn diện. Nó hoạt động ở tầng container, cung cấp tính năng giống như PaaS như triển khai, nhân rộng, cân bằng tải, nhưng là một giải pháp linh hoạt và có thể mở rộng, không giới hạn loại ứng dụng được hỗ trợ, không triển khai mã nguồn hoặc build ứng dụng, không cung cấp dịch vụ ứng dụng cấp cao như middleware, databases, không bắt buộc sử dụng các giải pháp ghi nhật ký, giám sát hoặc cảnh báo, và không cung cấp hoặc áp dụng bất kỳ cấu hình toàn diện, bảo trì, quản lý hoặc hệ thống tự phục hồi. Kubernetes loại bỏ nhu cầu về điều phối truyền thống, thay vào đó là kiểm soát liên tục từ trạng thái hiện tại sang trạng thái mong muốn.
