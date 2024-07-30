---
title: "Cách hoạt động"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: "<b> 3.1.3 </b>"
---

Trong Kubernetes, Pods có thể được gán ưu tiên so với các Pods khác. Lập lịch Kubernetes sẽ sử dụng điều này để ưu tiên Pods khác có ưu tiên thấp hơn để phù hợp với các Pods có ưu tiên cao hơn. `PriorityClass` là tài nguyên với các giá trị ưu tiên được tạo ra và gán cho Pods, và một `PriorityClass` mặc định có thể được gán cho một namespace.

Dưới đây là một ví dụ về một lớp ưu tiên cho phép một Pod có ưu tiên cao hơn so với các Pods khác:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
description: "Lớp ưu tiên được sử dụng cho các Pods có ưu tiên cao."
```

Dưới đây là một ví dụ về đặc tả Pod sử dụng lớp ưu tiên ở trên:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
    - name: nginx
      image: nginx
      imagePullPolicy: IfNotPresent
  priorityClassName: high-priority # Đặc tả Lớp Ưu Tiên được chỉ định
```

Tài liệu cho [Ưu Tiên và Sự Ưu Tiên Của Pods](https://kubernetes.io/docs/concepts/scheduling-eviction/Pod-priority-preemption/) giải thích cách hoạt động này chi tiết.

Làm thế nào chúng ta có thể áp dụng điều này để thực hiện việc tăng cường tài nguyên trong cụm EKS của chúng ta?

- Một lớp ưu tiên với giá trị ưu tiên **“-1"** được tạo ra và gán cho Pods [Pause Container](https://www.ianlewis.org/en/almighty-pause-container) trống. Các container "pause" trống hoạt động như các vị trí trống.

- Một lớp ưu tiên mặc định được tạo ra với giá trị ưu tiên **“0”.** Điều này được gán toàn cầu cho một cụm, vì vậy bất kỳ triển khai nào không có lớp ưu tiên sẽ được gán lớp ưu tiên mặc định này.

- Khi một công việc thực sự được lập lịch, các container trống sẽ bị loại bỏ và các Pods ứng dụng sẽ được cung cấp ngay lập tức.

- Khi có các Pods **Pending** (Pause Container) trong cụm của chúng ta, Cluster Autoscaler sẽ bắt đầu và cung cấp thêm các nút làm việc Kubernetes dựa trên cấu hình **ASG (`--max-size`)** được liên kết với nhóm nút EKS.

Mức độ tăng cường cần thiết có thể được kiểm soát bằng cách:

1. Số lượng Pause Pods (**replicas**) với các yêu cầu tài nguyên **CPU và bộ nhớ** cần thiết
2. Số lượng tối đa các nút trong nhóm nút EKS (`maxsize`)