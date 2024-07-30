---
title: "Cách hoạt động"
date: "`r Sys.Date()`"
weight: 4
chapter: false
pre: "<b> 3.1.4 </b>"
---


#### Tạo PriorityClass cho Ứng dụng trên AWS và Kubernetes

#### Tạo Default PriorityClass Toàn cầu

Trong quy trình tốt nhất, việc tạo `PriorityClass` phù hợp cho các ứng dụng của bạn được coi là quan trọng. Bây giờ, chúng ta sẽ tạo một default `PriorityClass` toàn cầu bằng cách sử dụng trường `globalDefault:true`. `PriorityClass` mặc định này sẽ được gán cho các pod/deployment không chỉ định `PriorityClassName`.

```yaml
manifests/modules/autoscaling/compute/overprovisioning/setup/priorityclass-default.yaml
```

#### Tạo PriorityClass cho Pause Pods

Chúng ta cũng sẽ tạo `PriorityClass` sẽ được gán cho các pause pod được sử dụng cho việc over-provisioning với giá trị ưu tiên `-1`.

```yaml
manifests/modules/autoscaling/compute/overprovisioning/setup/priorityclass-pause.yaml
```

Các pause pod đảm bảo rằng có đủ node sẵn có dựa trên mức over-provisioning cần thiết cho môi trường của bạn. Hãy nhớ đến tham số `—max-size` trong ASG (của nhóm node EKS). Cluster Autoscaler sẽ không tăng số lượng node vượt quá giới hạn tối đa được chỉ định trong ASG này.

```yaml
manifests/modules/autoscaling/compute/overprovisioning/setup/deployment-pause.yaml
```

Trong trường hợp này, chúng ta sẽ lên lịch cho một pause pod duy nhất yêu cầu `7Gi` bộ nhớ, điều này có nghĩa là nó sẽ tiêu thụ gần như một toàn bộ instance `m5.large`. Điều này sẽ dẫn đến việc luôn có 2 node "dư" sẵn có.

#### Áp dụng cập nhật cho cluster của bạn

```bash timeout=340 hook=overprovisioning-setup
$ kubectl apply -k ~/environment/eks-workshop/modules/autoscaling/compute/overprovisioning/setup
priorityclass.scheduling.k8s.io/default created
priorityclass.scheduling.k8s.io/pause-pods created
deployment.apps/pause-pods created
$ kubectl rollout status -n other deployment/pause-pods --timeout 300s
```

Khi quá trình này hoàn thành, các pause pods sẽ được chạy:

```bash
$ kubectl get pods -n other
NAME                          READY   STATUS    RESTARTS   AGE
pause-pods-7f7669b6d7-v27sl   1/1     Running   0          5m6s
pause-pods-7f7669b6d7-v7hqv   1/1     Running   0          5m6s
```

Và chúng ta có thể thấy các node bổ sung đã được cung cấp bởi CA:

```bash
$ kubectl get nodes -l workshop-default=yes
NAME                                         STATUS   ROLES    AGE     VERSION
ip-10-42-10-159.us-west-2.compute.internal   Ready    <none>   3d      vVAR::KUBERNETES_NODE_VERSION
ip-10-42-10-111.us-west-2.compute.internal   Ready    <none>   33s     vVAR::KUBERNETES_NODE_VERSION
ip-10-42-10-133.us-west-2.compute.internal   Ready    <none>   33s     vVAR::KUBERNETES_NODE_VERSION
ip-10-42-11-143.us-west-2.compute.internal   Ready    <none>   3d      vVAR::KUBERNETES_NODE_VERSION
ip-10-42-11-81.us-west-2.compute.internal    Ready    <none>   3d      vVAR::KUBERNETES_NODE_VERSION
ip-10-42-12-152.us-west-2.compute.internal   Ready    <none>   3m11s   vVAR::KUBERNETES_NODE_VERSION
```

Hai node này không chạy bất kỳ workloads nào ngoại trừ các pause pods của chúng tôi, những pods này sẽ bị trục xuất khi workloads "thực sự" được lên lịch.

