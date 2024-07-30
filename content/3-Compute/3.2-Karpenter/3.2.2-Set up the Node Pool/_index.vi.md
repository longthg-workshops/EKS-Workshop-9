---
title: "Cài đặt Node Pool"
date: "2024-04-03"
weight: 2
chapter: false
pre: "<b> 3.2.2 </b>"
---

#### Cài đặt Node Pool

Trong môi trường đám mây hiện đại, việc quản lý tài nguyên máy chủ một cách hiệu quả và tự động hóa là một trong những thách thức lớn nhất. Để giải quyết vấn đề này, nhiều công cụ và giải pháp đã được phát triển, trong đó có Karpenter - một công cụ quản lý tài nguyên phổ quát và mạnh mẽ dành cho Kubernetes.

#### Giới thiệu về Karpenter

Karpenter là một công cụ quản lý tài nguyên tự động cho Kubernetes, được xây dựng để giúp đơn giản hóa việc quản lý capacity trong môi trường đám mây. Trong bài viết này, chúng ta sẽ tìm hiểu về cách cấu hình Karpenter và bắt đầu triển khai các tài nguyên cơ bản để bắt đầu tự động hóa quy trình quản lý capacity trên nền tảng AWS.

#### Cấu hình Karpenter

Cấu hình của Karpenter được thực hiện thông qua `NodePool` CRD (Custom Resource Definition). Một `NodePool` Karpenter có khả năng xử lý nhiều hình dạng Pod khác nhau và đưa ra quyết định lập lịch và cung cấp dựa trên các thuộc tính của Pod như các nhãn (labels) và sự hảo hợp (affinity). Một cluster có thể có nhiều hơn một `NodePool`, nhưng hiện tại chúng ta sẽ chỉ khai báo một `NodePool` mặc định.

Một trong những mục tiêu chính của Karpenter là đơn giản hóa việc quản lý capacity. Nếu bạn quen thuộc với các giải pháp tự động mở rộng khác, bạn có thể đã nhận thấy rằng Karpenter có một cách tiếp cận khác, được gọi là **group-less auto scaling**. Karpenter cho phép chúng ta tránh sự phức tạp phát sinh từ việc quản lý nhiều loại ứng dụng với nhu cầu tính toán khác nhau.

#### Triển khai NodePool và EC2NodeClass

Để bắt đầu triển khai Karpenter, chúng ta cần áp dụng hai CRDs: `NodePool` và `EC2NodeClass`. Đây là yêu cầu cơ bản để Karpenter có thể xử lý các yêu cầu tự động hóa cơ bản.

**Cấu hình NodePool (nodepool.yaml)**:

```yaml
file
manifests/modules/autoscaling/compute/karpenter/nodepool/nodepool.yaml
```

:::info
Chúng ta yêu cầu NodePool bắt đầu tất cả các node mới với nhãn `type: karpenter`, điều này sẽ cho phép chúng ta chỉ định các node Karpenter cụ thể cho các Pod mục đích minh họa.
:::

#### Cấu hình NodePool và EC2NodeClass

Cấu hình của Karpenter được chia thành hai phần. Phần đầu tiên định nghĩa mô tả NodePool chung. Phần thứ hai được định nghĩa bởi cài đặt của nhà cung cấp cho AWS, trong trường hợp của chúng ta là `EC2NodeClass` và cung cấp cấu hình cụ thể áp dụng cho AWS. Cấu hình NodePool cụ thể này khá đơn giản, nhưng trong workshop chúng ta sẽ tùy chỉnh nó thêm. Hiện tại, hãy tập trung vào một số cài đặt được sử dụng.

- **Mục Yêu Cầu (Requirements Section)**: NodePool CRD hỗ trợ định nghĩa các thuộc tính của node như loại instance và zone. Trong ví dụ này, chúng ta đang thiết lập `karpenter.sh/capacity-type` để giới hạn ban đầu Karpenter chỉ cung cấp các instance On-Demand, cũng như `kubernetes.io/os`, `karpenter.k8s.aws/instance-category` và `karpenter.k8s.aws/instance-generation` để giới hạn một phần của loại instance phù hợp. Bạn có thể tìm hiểu các thuộc tính khác [tại đây](https://karpenter.sh/docs/concepts/scheduling/#selecting-nodes). Chúng ta sẽ làm việc với một số thuộc tính khác trong quá trình workshop.
- **Mục Giới Hạn (Limits section)**: NodePool có thể định nghĩa một giới hạn về lượng CPU và bộ nhớ được quản lý bởi nó. Khi giới hạn này đạt được, Karpenter sẽ không cung cấp thêm capacity liên kết với NodePool cụ thể đó, tạo ra một giới hạn về tổng compute.
- **Tags**: `EC2NodeClass` cũng có thể định nghĩa một tập hợp các tags mà các instance EC2 sẽ có sau khi được tạo ra. Điều này giúp kích hoạt quy trình kế toán và quản trị tại cấp độ EC2.
- **Selectors

**: Tài nguyên `EC2NodeClass` sử dụng `securityGroupSelectorTerms` và `subnetSelectorTerms` để tìm kiếm các tài nguyên được

 sử dụng để khởi chạy node. Những tags này được tự động thiết lập trên cơ sở hạ tầng AWS liên kết được cung cấp cho workshop.

#### Áp dụng NodePool và EC2NodeClass

Áp dụng `NodePool` và `EC2NodeClass` với lệnh sau:

```bash timeout=180
$ kubectl kustomize ~/environment/eks-workshop/modules/autoscaling/compute/karpenter/nodepool \
  | envsubst | kubectl apply -f-
```

Trong suốt workshop, bạn có thể kiểm tra log của Karpenter với lệnh sau để hiểu cách hoạt động của nó:

```bash
$ kubectl logs -l app.kubernetes.io/instance=karpenter -n karpenter | jq '.'
```

Karpenter mang lại một cách tiếp cận đơn giản và hiệu quả để tự động hóa quản lý capacity trong môi trường Kubernetes trên AWS. Bằng cách sử dụng Karpenter, bạn có thể tối ưu hóa việc sử dụng tài nguyên, giảm thiểu chi phí và tăng tính khả dụng của hệ thống. Hãy bắt đầu triển khai và tận dụng các lợi ích của Karpenter ngay hôm nay!

