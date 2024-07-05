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