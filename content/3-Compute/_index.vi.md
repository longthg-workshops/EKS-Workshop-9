---
title: "Compute"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 3. </b>"
---

#### Compute
Trong Kubernetes, một trong những khía cạnh đầu tiên mà chúng ta muốn đảm bảo tự động điều chỉnh là cơ sở hạ tầng tính toán EC2 được sử dụng để chạy các pods của chúng ta. Điều này sẽ điều chỉnh số lượng các instance EC2 có sẵn cho cụm EKS như là các worker nodes một cách linh hoạt khi các pods được thêm hoặc loại bỏ.

Có nhiều cách để triển khai tự động điều chỉnh quy mô tính toán trong Kubernetes, và tại AWS có hai cơ chế chính được sử dụng:

- Công cụ Kubernetes Cluster Autoscaler
- Karpenter

Trong chương này, chúng ta sẽ khám phá các cách khác nhau để đạt được tự động điều chỉnh quy mô tính toán trong Kubernetes trên AWS bằng cách sử dụng công cụ Kubernetes Cluster Autoscaler và cơ chế Karpenter.
