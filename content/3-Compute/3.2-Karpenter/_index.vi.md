---
title: "Karpenter"
date: "`r Sys.Date()`"
weight: 2
chapter: false
pre: "<b> 3.2 </b>"
---

#### Karpenter

Chuẩn bị môi trường cho phần này:

```bash timeout=900 wait=30
$ prepare-environment autoscaling/compute/karpenter
```

Điều này sẽ thực hiện những thay đổi sau vào môi trường thí nghiệm của bạn:

- Cài đặt Karpenter trong cụm Amazon EKS


Trong phần thực hành này, chúng ta sẽ tìm hiểu về [Karpenter](https://github.com/aws/karpenter), một dự án tự động co giãn mã nguồn mở được xây dựng cho Kubernetes. Karpenter được thiết kế để cung cấp tài nguyên máy tính phù hợp với nhu cầu của ứng dụng của bạn trong vài giây, không phải vài phút, bằng cách quan sát các yêu cầu tài nguyên tổng hợp của các pod không thể lập lịch và đưa ra quyết định để khởi chạy và kết thúc các nút để giảm thiểu độ trễ lập lịch.

![Sơ đồ Karpenter](./assets/karpenter-diagram.png)

Mục tiêu của Karpenter là cải thiện hiệu suất và chi phí của việc chạy các khối công việc trên các cụm Kubernetes. Karpenter hoạt động bằng cách:

- Theo dõi các pod mà lập lịch Kubernetes đã đánh dấu là không thể lập lịch
- Đánh giá các ràng buộc lập lịch (yêu cầu tài nguyên, nodeselectors, affinities, tolerations, và topology spread constraints) được yêu cầu bởi các pod
- Cung cấp các nút đáp ứng các yêu cầu của các pod
- Lập lịch các pod để chạy trên các nút mới
- Loại bỏ các nút khi các nút không còn cần thiết nữa
