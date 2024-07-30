---
title: "Cách hoạt động"
date: "`r Sys.Date()`"
weight: 5
chapter: false
pre: "<b> 3.1.5 </b>"
---

Trong bài thực hành này, chúng ta sẽ mở rộng kiến trúc ứng dụng của mình hơn so với phần CA và xem xét sự khác biệt về khả năng phản hồi.

```file
manifests/modules/autoscaling/compute/overprovisioning/scale/deployment.yaml
```

Áp dụng các cập nhật vào cluster của bạn:

```bash timeout=180 hook=overprovisioning-scale
$ kubectl apply -k ~/environment/eks-workshop/modules/autoscaling/compute/overprovisioning/scale
$ kubectl wait --for=condition=Ready --timeout=180s pods -l app.kubernetes.io/created-by=eks-workshop -A
```

Khi các pod mới triển khai, cuối cùng sẽ có một xung đột nơi các pod tạm dừng đang tiêu thụ tài nguyên mà các dịch vụ công việc có thể sử dụng. Do cấu hình ưu tiên của chúng tôi, pod tạm dừng sẽ bị đẩy ra để cho phép các pod công việc bắt đầu. Điều này sẽ để lại một số hoặc tất cả các pod tạm dừng ở trạng thái `Pending`:

```bash
$ kubectl get pod -n other -l run=pause-pods
NAME                          READY   STATUS    RESTARTS   AGE
pause-pods-5556d545f7-2pt9g   0/1     Pending   0          16m
pause-pods-5556d545f7-k5vj7   0/1     Pending   0          16m
```

Điều này sẽ cho phép các pod công việc của chúng tôi chuyển đổi sang trạng thái `ContainerCreating` và `Running` nhanh hơn.