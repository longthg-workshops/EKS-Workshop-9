---
title: "Scale with CA"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: "<b> 3.1.1 </b>"
---

#### Triển khai Cluster Autoscaler trên Amazon EKS

Trong khóa thực hành này, chúng ta đã cài đặt thành phần Cluster Autoscaler vào trong cụm EKS của chúng ta như một phần của việc chuẩn bị cho bài lab này. Nó chạy như một `deployment` trong namespace `kube-system`:

```bash
$ kubectl get deployment -n kube-system cluster-autoscaler-aws-cluster-autoscaler
```

Trong bài lab này, chúng ta sẽ cập nhật tất cả các thành phần ứng dụng để tăng số lượng replica lên 4. Điều này sẽ làm tăng việc sử dụng tài nguyên hơn so với những gì có sẵn trong một cụm, kích hoạt việc cung cấp thêm tài nguyên tính toán.

```file
manifests/modules/autoscaling/compute/cluster-autoscaler/deployment.yaml
```

Hãy áp dụng điều này vào cụm của chúng ta:

```bash hook=ca-pod-scaleout timeout=180
$ kubectl apply -k ~/environment/eks-workshop/modules/autoscaling/compute/cluster-autoscaler
```

Một số pod sẽ ở trạng thái `Pending`, điều này kích hoạt cluster-autoscaler để mở rộng fllet EC2.

```bash test=false
$ kubectl get pods -n orders -o wide --watch
```

Xem nhật ký của cluster-autoscaler

```bash test=false
$ kubectl -n kube-system logs \
  -f deployment/cluster-autoscaler-aws-cluster-autoscaler
```

Kiểm tra [EC2 AWS Management Console](https://console.aws.amazon.com/ec2/home?#Instances:sort=instanceId) để xác nhận rằng các nhóm Auto Scaling đang mở rộng để đáp ứng nhu cầu. Điều này có thể mất vài phút. Bạn cũng có thể theo dõi quá trình triển khai pod từ dòng lệnh. Bạn sẽ thấy các pod chuyển từ trạng thái pending sang running khi các node được mở rộng.

Hoặc bạn có thể sử dụng `kubectl`:

```bash
$ kubectl get nodes -l workshop-default=yes
NAME                                         STATUS   ROLES    AGE     VERSION
ip-10-42-10-159.us-west-2.compute.internal   Ready    <none>   3d      vVAR::KUBERNETES_NODE_VERSION
ip-10-42-11-143.us-west-2.compute.internal   Ready    <none>   3d      vVAR::KUBERNETES_NODE_VERSION
ip-10-42-11-81.us-west-2.compute.internal    Ready    <none>   3d      vVAR::KUBERNETES_NODE_VERSION
ip-10-42-12-152.us-west-2.compute.internal   Ready    <none>   3m11s   vVAR::KUBERNETES_NODE_VERSION
```

---
Lưu ý: `VAR::KUBERNETES_NODE_VERSION` là biến dùng để đại diện cho phiên bản Kubernetes của node.