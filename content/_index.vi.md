---
title: "Tự Động Mở Rộng trên AWS với Kubernetes"
date: "`r Sys.Date()`"
weight: 1
chapter: false
---

# Tự Động Mở Rộng trên AWS với Kubernetes

Tự động mở rộng giám sát các khối công việc của bạn và tự động điều chỉnh dung lượng để duy trì hiệu suất ổn định, dự đoán được đồng thời tối ưu hóa chi phí. Khi sử dụng Kubernetes, có hai cơ chế chính có liên quan mà có thể được sử dụng để tự động mở rộng:

- **Compute (Tính toán):** Khi các pods được mở rộng, tính toán cơ bản trong một cụm Kubernetes cũng phải thích ứng bằng cách điều chỉnh số lượng hoặc kích thước của các nút worker được sử dụng để chạy các Pods.
- **Pods:** Vì các pods được sử dụng để chạy các khối công việc trong một cụm Kubernetes, việc mở rộng một khối công việc chủ yếu được thực hiện bằng cách mở rộng Pods theo chiều ngang hoặc theo chiều dọc để phản ứng với các tình huống như thay đổi tải trên một ứng dụng cụ thể.

Trong chương này, chúng ta sẽ khám phá các cơ chế khác nhau có sẵn để tự động mở rộng cả số lượng pods và khả năng tính toán của một cụm.