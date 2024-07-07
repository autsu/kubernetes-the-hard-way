# pod 的 CIDR 在哪里配置？

从本项目看，在 kube-proxy-config 中定义了 pod 的 CIDR，如下：

```yaml
clusterCIDR: "10.200.0.0/16"
```

以及 kube-controller-manager 的启动参数

```text
  --cluster-cidr=10.200.0.0/16 \
```

[源码分析 Kubernetes 对 Pod IP 的管理 – 陈少文的网站](https://www.chenshaowen.com/blog/source-analysis-kubernetes-management-of-pod-ip.html)


# kubectl get node 只显示 worker 节点，且 Roles 都为 <none>

似乎是因为本教程中只给 worker 节点部署了 kubelet，而想让 get node 显示 master 节点，也需要在 master 节点上部署 kubelet，然后 Kubelet 注册 apiserver 后
就能显示了

至于 Roles 为 none，应该是需要手动去打 label