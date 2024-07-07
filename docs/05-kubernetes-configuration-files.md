# Generating Kubernetes Configuration Files for Authentication

In this lab you will generate [Kubernetes configuration files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/), also known as kubeconfigs, which enable Kubernetes clients to locate and authenticate to the Kubernetes API Servers.

在本实验中，您将生成 [Kubernetes 配置文件](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)，也称为 kubeconfigs，它使 Kubernetes 客户端能够定位并进行身份验证Kubernetes API 服务器。

## Client Authentication Configs

In this section you will generate kubeconfig files for the `kubelet` and the `admin` user.

在本节中，您将为“kubelet”和“admin”用户生成 kubeconfig 文件。

### The kubelet Kubernetes Configuration File

When generating kubeconfig files for Kubelets the client certificate matching the Kubelet's node name must be used. This will ensure Kubelets are properly authorized by the Kubernetes [Node Authorizer](https://kubernetes.io/docs/admin/authorization/node/).

为 Kubelet 生成 kubeconfig 文件时，**必须使用与 Kubelet 节点名称匹配的客户端证书**。这将确保 Kubelet 得到 Kubernetes 节点授权者的正确授权。

> The following commands must be run in the same directory used to generate the SSL certificates during the [Generating TLS Certificates](04-certificate-authority.md) lab.

Generate a kubeconfig file the node-0 worker node:

给工作节点生成 kubeconfig 文件

> 这段命令的作用是为每个节点（在这个例子中是`node-0`和`node-1`）生成配置文件`kubeconfig`，以便它们能够安全地与Kubernetes集群通信。以下是这段脚本做的事情分解：
>
> 1. 通过循环`for host in node-0 node-1; do ... done`，这段脚本对`node-0`和`node-1`这两个节点执行相同的操作。
>
> 2. 为每个节点设置一个指向集群的`kubeconfig`，其中包含了集群的名称(`kubernetes-the-hard-way`)、集群的API服务器地址(`https://server.kubernetes.local:6443`)、和证书颁发机构的证书(`ca.crt`)。使用`--embed-certs=true`的选项把证书内容直接嵌入到`kubeconfig`文件中，避免需要在文件系统中保存单独的证书文件。
>
>    ```sh
>    kubectl config set-cluster kubernetes-the-hard-way \
>      --certificate-authority=ca.crt \
>      --embed-certs=true \
>      --server=https://server.kubernetes.local:6443 \
>      --kubeconfig=${host}.kubeconfig
>    ```
>
> 3. 为每个节点设置用户凭据，用户的名称遵循`system:node:${host}`的格式（例如`system:node:node-0`），并使用节点的客户端证书(`${host}.crt`)和私钥(`${host}.key`)。与集群设置相同，使用`--embed-certs=true`将证书直接嵌入到`kubeconfig`。
>
>    ```sh
>    kubectl config set-credentials system:node:${host} \
>      --client-certificate=${host}.crt \
>      --client-key=${host}.key \
>      --embed-certs=true \
>      --kubeconfig=${host}.kubeconfig
>    ```
>
> 4. 为每个节点配置默认上下文，将之前定义的集群和用户（节点作为用户）关联起来。上下文是`kubectl`管理Kubernetes资源时使用的集群、用户和命名空间的组合。
>
>    ```sh
>    kubectl config set-context default \
>      --cluster=kubernetes-the-hard-way \
>      --user=system:node:${host} \
>      --kubeconfig=${host}.kubeconfig
>    ```
>
> 5. 激活使用这个默认上下文。这意味着当使用此`kubeconfig`文件时，`kubectl`命令将默认使用指定的集群、用户和命名空间。
>
>    ```sh
>    kubectl config use-context default \
>      --kubeconfig=${host}.kubeconfig
>    ```
>
> 总结来说，这段脚本针对每个指定的节点，自动创建了一个包含了必要证书、指向特定Kubernetes集群的配置（`kubeconfig`）。这使得每个节点都能够以其所分配的身份证书来安全地与集群通信。



```bash
for host in node-0 node-1; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=${host}.kubeconfig

  kubectl config set-credentials system:node:${host} \
    --client-certificate=${host}.crt \
    --client-key=${host}.key \
    --embed-certs=true \
    --kubeconfig=${host}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${host} \
    --kubeconfig=${host}.kubeconfig

  kubectl config use-context default \
    --kubeconfig=${host}.kubeconfig
done
```

Results:

会生成如下两个文件

```text
node-0.kubeconfig
node-1.kubeconfig
```

### The kube-proxy Kubernetes Configuration File

Generate a kubeconfig file for the `kube-proxy` service:

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.crt \
    --client-key=kube-proxy.key \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-proxy.kubeconfig
}
```

Results:

```text
kube-proxy.kubeconfig
```

### The kube-controller-manager Kubernetes Configuration File

Generate a kubeconfig file for the `kube-controller-manager` service:

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.crt \
    --client-key=kube-controller-manager.key \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-controller-manager.kubeconfig
}
```

Results:

```text
kube-controller-manager.kubeconfig
```


### The kube-scheduler Kubernetes Configuration File

Generate a kubeconfig file for the `kube-scheduler` service:

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://server.kubernetes.local:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.crt \
    --client-key=kube-scheduler.key \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default \
    --kubeconfig=kube-scheduler.kubeconfig
}
```

Results:

```text
kube-scheduler.kubeconfig
```

### The admin Kubernetes Configuration File

Generate a kubeconfig file for the `admin` user:

```bash
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.crt \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.crt \
    --client-key=admin.key \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default \
    --kubeconfig=admin.kubeconfig
}
```

Results:

```text
admin.kubeconfig
```

## Distribute the Kubernetes Configuration Files

Copy the `kubelet` and `kube-proxy` kubeconfig files to the node-0 instance:

```bash
for host in node-0 node-1; do
  ssh root@$host "mkdir /var/lib/{kube-proxy,kubelet}"
  
  scp kube-proxy.kubeconfig \
    root@$host:/var/lib/kube-proxy/kubeconfig \
  
  scp ${host}.kubeconfig \
    root@$host:/var/lib/kubelet/kubeconfig
done
```

我的

```shell
{
	mkdir -p /var/lib/{kube-proxy,kubelet}
	cp kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
	cp node-0.kubeconfig /var/lib/kubelet/kubeconfig
}

{
	mkdir -p /var/lib/{kube-proxy,kubelet}
	cp kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
	cp node-1.kubeconfig /var/lib/kubelet/kubeconfig
}
```



Copy the `kube-controller-manager` and `kube-scheduler` kubeconfig files to the controller instance:

```bash
scp admin.kubeconfig \
  kube-controller-manager.kubeconfig \
  kube-scheduler.kubeconfig \
  root@server:~/
```

Next: [Generating the Data Encryption Config and Key](06-data-encryption-keys.md)
