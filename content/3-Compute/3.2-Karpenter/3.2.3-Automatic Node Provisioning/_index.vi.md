---
title: "Tự Động Cấp Phát Node"
date: "2024-04-03"
weight: 3
chapter: false
pre: "<b> 3.2.3 </b>"
---

#### Tự Động Cấp Phát Node

Trong phần này, chúng ta sẽ khám phá cách Karpenter có thể tự động cấp phát các máy EC2 có kích thước phù hợp dựa trên nhu cầu của các Pods không thể được lên lịch vào bất kỳ thời điểm nào. Điều này giúp giảm lượng tài nguyên tính toán không sử dụng trong một cụm EKS.

#### Các Loại Instance

Trước hết, hãy xem xét các loại instance mà Karpenter có thể sử dụng:

| Loại Instance | vCPU | Bộ Nhớ | Giá   |
| ------------- | ---- | ------ | ----- |
| c5.large      | 2    | 4GB    | +     |
| m5.large      | 2    | 8GB    | ++    |
| r5.large      | 2    | 16GB   | +++   |
| m5.xlarge     | 4    | 16GB   | ++++  |

#### Tạo Các Pods và Quan Sát Karpenter

Hiện tại, không có node nào được quản lý bởi Karpenter:

```bash
$ kubectl get node -l type=karpenter
Không tìm thấy tài nguyên
```

Hãy tạo một số Pods và quan sát cách Karpenter thích ứng. Hiện tại, vẫn chưa có bất kỳ node nào được quản lý bởi Karpenter:

```yaml
manifests/modules/autoscaling/compute/karpenter/scale/deployment.yaml
```

Ứng dụng `Deployment` dưới đây sử dụng một `pause` container image đơn giản với yêu cầu tài nguyên là `memory: 1Gi`, sẽ được sử dụng để mở rộng cụm một cách dễ dàng. Lưu ý rằng `nodeSelector` được đặt thành `type: karpenter`, yêu cầu các Pods chỉ được lên lịch trên các node có nhãn đó được cấp phát bởi Karpenter `NodePool`. Bắt đầu với `0` bản sao:

```yaml
manifests/modules/autoscaling/compute/karpenter/scale/deployment.yaml
```

:::info Pause container là gì?
Trong ví dụ này, bạn sẽ thấy chúng tôi sử dụng hình ảnh:

`public.ecr.aws/eks-distro/kubernetes/pause`

Đây là một container nhỏ không sử dụng bất kỳ tài nguyên thực sự nào và bắt đầu nhanh chóng, điều này làm cho nó trở nên tuyệt vời cho việc minh họa các kịch bản mở rộng. Chúng tôi sẽ sử dụng nó cho nhiều ví dụ trong lab cụ thể này.
:::

Áp dụng Deployment này:

```bash
$ kubectl apply -k ~/environment/eks-workshop/modules/autoscaling/compute/karpenter/scale
deployment.apps/inflate đã được tạo
```

Bây giờ hãy mở rộng Deployment này một cách cố ý để minh họa rằng Karpenter đang đưa ra quyết định tối ưu. Vì chúng ta đã yêu cầu 1Gi bộ nhớ, nếu chúng ta mở rộng Deployment lên 5 bản sao thì sẽ yêu cầu tổng cộng 5Gi bộ nhớ.

Trước khi tiếp tục, bạn nghĩ Karpenter sẽ cấp phát loại instance nào từ bảng trên? Bạn muốn nó chọn loại instance nào?

Mở rộng Deployment:

```bash
$ kubectl scale -n other deployment/inflate --replicas 5
```

Do thao tác này tạo ra một hoặc nhiều máy EC2 mới nên sẽ mất một chút thời gian, bạn có thể sử dụng `kubectl` để chờ đợi cho đến khi nó hoàn thành với lệnh này:

```bash hook=karpenter-deployment
$ kubectl rollout status -n other deployment/inflate --timeout=180s
```

Khi tất cả các Pods đều đang chạy, hãy xem Karpenter đã chọn loại instance nào:

```bash
$ kubectl logs -l app.kubernetes.io/instance=karpenter -n karpenter | grep 'launched nodeclaim' | jq '.'
```

Bạn sẽ thấy đầu ra cho biết loại instance và tùy chọn mua:

```json
{
  "level": "INFO",
  "time": "2023-11-16T22:32:00.413Z",
  "logger": "controller.nodeclaim.lifecycle",
  "message": "launched nodeclaim",
  "commit": "1072d3b",
  "nodeclaim": "default-xxm79",
  "nodepool": "default",
  "provider-id": "aws:///us-west-2a/i-0bb8a7e6111d45591",
  # ĐIỂM NỔI BẬT
  "instance-type": "m5.large",
  "zone": "us-west-2a",
  # ĐIỂM NỔI BẬT
  "capacity-type": "on-demand",
  "allocatable": {
    "cpu": "1930m",
    "ephemeral-storage": "17Gi",
    "memory": "6903Mi",
    "pods": "29",
    "vpc.amazonaws.com/pod-eni": "9"
  }
}
```

Các Pods mà chúng ta đã lên lịch sẽ phù hợp hoàn hảo với một loại instance EC2 có 8GB bộ nhớ, và vì Karpenter luôn ưu tiên loại instance có giá thấp nhất cho các instance on-demand, n

ó sẽ chọn `m5.large`.

:::info
Có những trường hợp nhất định mà một loại instance khác có thể được chọn thay vì loại giá thấp nhất, ví dụ nếu loại instance rẻ nhất đó không còn sẵn dung lượng nữa trong khu vực bạn đang làm việc.
:::

Chúng ta cũng có thể kiểm tra các siêu dữ liệu được thêm vào node bởi Karpenter:

```bash
$ kubectl get node -l type=karpenter -o jsonpath='{.items[0].metadata.labels}' | jq '.'
```

Đầu ra này sẽ hiển thị các nhãn khác nhau đã được đặt, ví dụ như loại instance, tùy chọn mua, vùng khả dụng và nhiều hơn nữa:

```json
{
  "beta.kubernetes.io/arch": "amd64",
  "beta.kubernetes.io/instance-type": "m5.large",
  "beta.kubernetes.io/os": "linux",
  "failure-domain.beta.kubernetes.io/region": "us-west-2",
  "failure-domain.beta.kubernetes.io/zone": "us-west-2a",
  "k8s.io/cloud-provider-aws": "1911afb91fc78905500a801c7b5ae731",
  "karpenter.k8s.aws/instance-category": "m",
  "karpenter.k8s.aws/instance-cpu": "2",
  "karpenter.k8s.aws/instance-family": "m5",
  "karpenter.k8s.aws/instance-generation": "5",
  "karpenter.k8s.aws/instance-hypervisor": "nitro",
  "karpenter.k8s.aws/instance-memory": "8192",
  "karpenter.k8s.aws/instance-pods": "29",
  "karpenter.k8s.aws/instance-size": "large",
  "karpenter.sh/capacity-type": "on-demand",
  "karpenter.sh/initialized": "true",
  "karpenter.sh/provisioner-name": "default",
  "kubernetes.io/arch": "amd64",
  "kubernetes.io/hostname": "ip-100-64-10-200.us-west-2.compute.internal",
  "kubernetes.io/os": "linux",
  "node.kubernetes.io/instance-type": "m5.large",
  "topology.ebs.csi.aws.com/zone": "us-west-2a",
  "topology.kubernetes.io/region": "us-west-2",
  "topology.kubernetes.io/zone": "us-west-2a",
  "type": "karpenter",
  "vpc.amazonaws.com/has-trunk-attached": "true"
}
```

Ví dụ đơn giản này minh họa cho việc Karpenter có thể chọn đúng loại instance dựa trên các yêu cầu tài nguyên của các công việc cần dung lượng tính toán. Điều này khác biệt một cách cơ bản so với một mô hình dựa trên các nhóm node, như Cluster Autoscaler, trong đó các loại instance trong một nhóm node duy nhất phải có các đặc điểm CPU và bộ nhớ nhất định.
