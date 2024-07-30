---
title: "Disruption"
date: "2024-04-03"
weight: 4
chapter: false
pre: "<b> 3.2.4 </b>"
---

#### Disruption

Trong môi trường **AWS (Amazon Web Services)** và **Kubernetes**, việc tối ưu hóa cơ sở hạ tầng tính toán là một phần quan trọng của việc giảm chi phí. **Karpenter** là một công cụ tự động hóa quản lý nguồn lực tính toán trên Kubernetes, giúp tối ưu hóa việc sử dụng tài nguyên và giảm chi phí. **Karpenter** tự động phát hiện các nút có thể gây ra sự gián đoạn và triển khai các nút thay thế khi cần thiết, và điều này có thể xảy ra với ba lý do khác nhau:

- **Hết hạn**: Mặc định, **Karpenter** tự động hết hạn các phiên bản sau 720 giờ (30 ngày), buộc phải tái tạo để giữ các nút được cập nhật.
- **Thay đổi**: **Karpenter** phát hiện ra các thay đổi trong cấu hình (như `NodePool` hoặc `EC2NodeClass`) để áp dụng các thay đổi cần thiết.
- **Tổng hợp**: Một tính năng quan trọng để vận hành tính toán một cách hiệu quả về chi phí, **Karpenter** sẽ tối ưu hóa tính toán của cụm của chúng ta một cách liên tục. Ví dụ, nếu các khối công việc đang chạy trên các phiên bản tính toán không được sử dụng hiệu quả, nó sẽ tổng hợp chúng thành ít hơn các phiên bản.

#### Cấu hình Gián đoạn

Sự gián đoạn được cấu hình thông qua khối `disruption` trong một `NodePool`. Chính sách sau đã được cấu hình trên `NodePool` của chúng ta mà chúng ta đã xác định trong một phần trước của lab này.

```yaml
disruption:
  consolidationPolicy: WhenUnderutilized
  expireAfter: 720h # 30 * 24h = 720h
```

`consolidationPolicy` cũng có thể được đặt thành `WhenEmpty`, giới hạn sự gián đoạn chỉ trên các nút không chứa bất kỳ pod công việc nào. Tìm hiểu thêm về Sự gián đoạn trên [tài liệu Karpenter](https://karpenter.sh/docs/concepts/disruption/).

#### Kích hoạt Tổng hợp Tự động

1. Mở rộng khối công việc `inflate` từ 5 đến 12 bản sao, kích hoạt Karpenter cung cấp khả năng chứa thêm.
2. Giảm khối công việc xuống còn 5 bản sao.
3. Quan sát Karpenter tổng hợp tính toán.

```bash
$ kubectl scale -n other deployment/inflate --replicas 12
$ kubectl rollout status -n other deployment/inflate --timeout=180s
```

Điều này thay đổi tổng yêu cầu bộ nhớ cho việc triển khai này khoảng 12Gi, điều này khi điều chỉnh để tính cả khoảng 600Mi dành cho kubelet trên mỗi nút có nghĩa là điều này sẽ vừa với 2 phiên bản của loại `m5.large`:

```bash
$ kubectl get nodes -l type=karpenter --label-columns node.kubernetes.io/instance-type
NAME                                         STATUS   ROLES    AGE     VERSION               INSTANCE-TYPE
ip-10-42-44-164.us-west-2.compute.internal   Ready    <none>   3m30s   vVAR::KUBERNETES_NODE_VERSION   m5.large
ip-10-42-9-102.us-west-2.compute.internal    Ready    <none>   14m     vVAR::KUBERNETES_NODE_VERSION   m5.large
```

Tiếp theo, giảm số lượng bản sao xuống còn 5:

```bash
$ kubectl scale -n other deployment/inflate --replicas 5
```

Chúng ta có thể kiểm tra nhật ký của **Karpenter** để có được một ý tưởng về các hành động mà nó đã thực hiện để đáp ứng việc mở rộng trong việc triển khai. Chờ khoảng 5-10 giây trước khi chạy lệnh sau:

```bash test=false
$ kubectl logs -l app.kubernetes.io/instance=karpenter -n karpenter | grep 'consolidation delete' | jq '.'
```

Kết quả sẽ cho thấy **Karpenter** xác định các nút cụ thể để gián đoạn, dòng nước và sau đó chấm dứt:

```json
{
  "level": "INFO",
  "time": "2023-11-16T22:47:05.659Z",
  "logger": "controller.disruption",
  "message": "disrupting via consolidation delete, terminating 1 candidates ip-10-42-44-164.us-west-2.compute.internal/m5.large/on-demand",
  "commit": "1072d3b"
}
```

Điều này sẽ dẫn đến lập lịch của Kubernetes đặt bất kỳ Pod nào trên các nút đó trên khả năng còn lại, và bây giờ chúng ta có thể thấy rằng **Karpenter** đang quản lý tổng cộng 1 nút:

```bash
$ kubectl get nodes -l type=karpenter
NAME                                         STATUS   ROLES    AGE     VERSION               INSTANCE-TYPE
ip-10

-42-44-164.us-west-2.compute.internal   Ready    <none>   6m30s   vVAR::KUBERNETES_NODE_VERSION   m5.large
```

**Karpenter** cũng có thể tổng hợp thêm nếu một nút có thể được thay thế bằng một biến thể rẻ hơn phản ứng lại với các thay đổi công việc. Điều này có thể được minh họa bằng cách giảm số lượng bản sao của triển khai `inflate` xuống còn 1, với tổng yêu cầu bộ nhớ khoảng 1Gi:

```bash
$ kubectl scale -n other deployment/inflate --replicas 1
```

Chúng ta có thể kiểm tra nhật ký của **Karpenter** và xem những hành động mà điều khiển đã thực hiện phản ứng:

```bash test=false
$ kubectl logs -l app.kubernetes.io/instance=karpenter -n karpenter -f | jq '.'
```

:::tip
Lệnh trước bao gồm cờ "-f" cho phép theo dõi, cho phép chúng ta xem nhật ký khi chúng xảy ra. Tổng hợp thành một nút nhỏ hơn mất ít hơn một phút. Theo dõi nhật ký để xem điều khiển Karpenter hoạt động như thế nào.
:::

Kết quả sẽ cho thấy **Karpenter** tổng hợp thông qua việc thay thế, thay thế nút m5.large bằng loại c5.large rẻ hơn được xác định trong Provisioner:

```json
{
  "level": "INFO",
  "time": "2023-11-16T22:50:23.249Z",
  "logger": "controller.disruption",
  "message": "disrupting via consolidation replace, terminating 1 candidates ip-10-42-9-102.us-west-2.compute.internal/m5.large/on-demand and replacing with on-demand node from types c5.large",
  "commit": "1072d3b"
}
```

Vì tổng yêu cầu bộ nhớ với 1 bản sao thấp hơn nhiều khoảng 1Gi, nó sẽ hiệu quả hơn khi chạy trên loại c5.large rẻ hơn với 4GB bộ nhớ. Khi nút được thay thế, chúng ta có thể kiểm tra siêu dữ liệu trên nút mới và xác nhận loại phiên bản là c5.large:

```bash
$ kubectl get nodes -l type=karpenter -o jsonpath="{range .items[*]}{.metadata.labels.node\.kubernetes\.io/instance-type}{'\n'}{end}"
c5.large
```

Đây là một ví dụ minh họa về cách **Karpenter** có thể tự động tối ưu hóa việc sử dụng tài nguyên và giảm chi phí trong môi trường AWS và Kubernetes. Sử dụng các tính năng của nó, chúng ta có thể duy trì một cơ sở hạ tầng tính toán hiệu quả và linh hoạt.