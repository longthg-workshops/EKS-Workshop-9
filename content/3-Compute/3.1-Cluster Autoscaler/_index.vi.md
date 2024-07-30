---
title: "Cluster Autoscaler"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 3.1 </b>"
---

#### Cluster Autoscaler

Chuẩn bị môi trường cho phần này:

```bash timeout=300 wait=30
$ prepare-environment autoscaling/compute/cluster-autoscaler
```

Điều này sẽ thực hiện các thay đổi sau vào môi trường lab của bạn:

- Cài đặt Kubernetes Cluster Autoscaler trong cụm Amazon EKS.

Trong bài lab này, chúng ta sẽ tìm hiểu về [Kubernetes Cluster Autoscaler](https://github.com/kubernetes/autoscaler), một thành phần tự động điều chỉnh kích thước của một Cụm Kubernetes để tất cả các pod có nơi để chạy mà không cần thiết phải có các nút không cần thiết. Cluster Autoscaler là một công cụ tuyệt vời để đảm bảo rằng cơ sở hạ tầng cụm dưới lying là linh hoạt, có thể mở rộng và có thể đáp ứng các yêu cầu thay đổi của công việc.

Kubernetes Cluster Autoscaler tự động điều chỉnh kích thước của một cụm Kubernetes khi một trong các điều kiện sau là đúng:

1. Có các pod không thể chạy trong một cụm do tài nguyên không đủ.
2. Có các nút trong một cụm không được sử dụng hiệu quả trong một khoảng thời gian kéo dài và các pod của chúng có thể được đặt trên các nút hiện có khác.

Cluster Autoscaler cho AWS cung cấp [tích hợp với các nhóm Tự động Mở rộng](https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler/cloudprovider/aws).

Trong bài lab này, chúng ta sẽ áp dụng Cluster Autoscaler vào cụm EKS của chúng ta và xem nó hoạt động như thế nào khi chúng ta mở rộng công việc của mình.