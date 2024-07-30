---
title: "Cấu hình Karpenter"
date: "2024-04-03"
weight: 1
chapter: false
pre: "<b> 3.2.1 </b>"
---

#### Cấu hình Karpenter

Trong dự án của chúng tôi, chúng tôi đã triển khai thành công Karpenter trên cluster EKS và chạy dưới dạng một deployment. Dưới đây là thông tin về trạng thái của deployment:

```bash
$ kubectl get deployment -n karpenter
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
karpenter   2/2     2            2           105s
```

Việc duy nhất chúng ta cần thực hiện là cập nhật các ánh xạ IAM của EKS để cho phép các node của Karpenter tham gia vào cluster:

```bash
$ eksctl create iamidentitymapping --cluster $EKS_CLUSTER_NAME \
    --region $AWS_REGION --arn $KARP_ARN \
    --group system:bootstrappers --group system:nodes \
    --username system:node:{{EC2PrivateDNSName}}
```

Qua đó, chúng ta đã cài đặt và cấu hình Karpenter thành công trên cluster EKS của mình. Điều này sẽ giúp tự động hóa việc quản lý tài nguyên và tăng khả năng mở rộng của hệ thống.
