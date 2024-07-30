---
title: "Cluster Over-Provisioning"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 3.1.2 </b>"
---

#### Cluster Over-Provisioning

Cluster Autoscaler (CA) cho AWS cấu hình nhóm tự động mở rộng EC2 (ASG) của EKS node group để tự động mở rộng các node trong cụm khi có các pod đang chờ được lên lịch.

Quá trình này của việc thêm các node vào một cụm bằng cách sửa đổi ASG ngầm hiện thêm thời gian phụ trước khi các pod có thể được lên lịch. Ví dụ, trong phần trước đó, bạn đã nhận thấy rằng mất vài phút trước khi các pod được tạo ra khi ứng dụng được mở rộng trở nên có sẵn.

Có nhiều cách tiếp cận để giải quyết vấn đề này. Bài tập thực hành này giải quyết vấn đề này bằng cách "thừa cấp" cụm với các node phụ cấp thêm chạy các pod ưu tiên thấp được sử dụng như các vị trí giữ chỗ. Các pod ưu tiên thấp này sẽ bị loại bỏ khi các pod ứng dụng quan trọng được triển khai. Các pod trống không chỉ đảm bảo rằng các tài nguyên CPU và bộ nhớ được dành trước mà còn đảm bảo rằng các địa chỉ IP được gán từ CNI AWS VPC Container Network Interface.