# Provisioning a CA and Generating TLS Certificates

In this lab you will provision a [PKI Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure) using openssl to bootstrap a Certificate Authority, and generate TLS certificates for the following components: kube-apiserver, kube-controller-manager, kube-scheduler, kubelet, and kube-proxy. The commands in this section should be run from the `jumpbox`.

## Certificate Authority

In this section you will provision a Certificate Authority that can be used to generate additional TLS certificates for the other Kubernetes components. Setting up CA and generating certificates using `openssl` can be time-consuming, especially when doing it for the first time. To streamline this lab, I've included an openssl configuration file `ca.conf`, which defines all the details needed to generate certificates for each Kubernetes component. 

Take a moment to review the `ca.conf` configuration file:

在本部分中，您将配置一个证书颁发机构，可用于为其他 Kubernetes 组件生成其他 TLS 证书。使用 openssl 设置 CA 并生成证书可能非常耗时，尤其是第一次执行此操作时。为了简化本实验，我添加了一个 openssl 配置文件 ca.conf，它定义了为每个 Kubernetes 组件生成证书所需的所有详细信息。

花点时间查看 ca.conf 配置文件：

```bash
cat ca.conf
```

You don't need to understand everything in the `ca.conf` file to complete this tutorial, but you should consider it a starting point for learning `openssl` and the configuration that goes into managing certificates at a high level.

Every certificate authority starts with a private key and root certificate. In this section we are going to create a self-signed certificate authority, and while that's all we need for this tutorial, this shouldn't be considered something you would do in a real-world production level environment. 

Generate the CA configuration file, certificate, and private key:

您不需要了解 ca.conf 文件中的所有内容来完成本教程，但您应该将其视为学习 openssl 以及高级管理证书的配置的起点。

每个证书颁发机构都以私钥和根证书开始。在本节中，我们将创建一个自签名证书颁发机构，虽然这就是本教程所需的全部内容，但这不应被视为您在实际生产级别环境中要做的事情。

生成CA配置文件、证书和私钥：

```bash
{
  openssl genrsa -out ca.key 4096
  openssl req -x509 -new -sha512 -noenc \
    -key ca.key -days 3653 \
    -config ca.conf \
    -out ca.crt
}
```

Results:

```txt
ca.crt ca.key
```

## Create Client and Server Certificates

In this section you will generate client and server certificates for each Kubernetes component and a client certificate for the Kubernetes `admin` user.

Generate the certificates and private keys:

在本部分中，您将为每个 Kubernetes 组件生成客户端和服务器证书，以及 Kubernetes 管理员用户的客户端证书。

生成证书和私钥：

```bash
certs=(
  "admin" "node-0" "node-1"
  "kube-proxy" "kube-scheduler"
  "kube-controller-manager"
  "kube-api-server"
  "service-accounts"
)
```

```bash
for i in ${certs[*]}; do
  openssl genrsa -out "${i}.key" 4096

  openssl req -new -key "${i}.key" -sha256 \
    -config "ca.conf" -section ${i} \
    -out "${i}.csr"
  
  openssl x509 -req -days 3653 -in "${i}.csr" \
    -copy_extensions copyall \
    -sha256 -CA "ca.crt" \
    -CAkey "ca.key" \
    -CAcreateserial \
    -out "${i}.crt"
done
```

The results of running the above command will generate a private key, certificate request, and signed SSL certificate for each of the Kubernetes components. You can list the generated files with the following command:

```bash
ls -1 *.crt *.key *.csr
```

生成内容如下：

```shell
admin.crt
admin.csr
admin.key
ca.crt
ca.key
kube-api-server.crt
kube-api-server.csr
kube-api-server.key
kube-controller-manager.crt
kube-controller-manager.csr
kube-controller-manager.key
kube-proxy.crt
kube-proxy.csr
kube-proxy.key
kube-scheduler.crt
kube-scheduler.csr
kube-scheduler.key
node-0.crt
node-0.csr
node-0.key
node-1.crt
node-1.csr
node-1.key
service-accounts.crt
service-accounts.csr
service-accounts.key
```



## Distribute the Client and Server Certificates

In this section you will copy the various certificates to each machine under a directory that each Kubernetes components will search for the certificate pair. In a real-world environment these certificates should be treated like a set of sensitive secrets as they are often used as credentials by the Kubernetes components to authenticate to each other.

Copy the appropriate certificates and private keys to the `node-0` and `node-1` machines:

在本部分中，您将把各种证书复制到每个计算机的目录下，每个 Kubernetes 组件将在该目录中搜索证书对。在现实环境中，这些证书应被视为一组敏感机密，因为 Kubernetes 组件经常将它们用作凭据来相互进行身份验证。

将相应的证书和私钥复制到 node-0 和 node-1 机器上：

```bash
for host in node-0 node-1; do
  ssh root@$host mkdir /var/lib/kubelet/
  
  scp ca.crt root@$host:/var/lib/kubelet/
    
  scp $host.crt \
    root@$host:/var/lib/kubelet/kubelet.crt
    
  scp $host.key \
    root@$host:/var/lib/kubelet/kubelet.key
done

```

orbstack 不支持虚拟机间 scp，但由于每台虚拟机都共享宿主机文件，所以可以在 mac 上下载好，然后虚拟机 cd 到该目录执行

在两个 node 节点执行：

node-0：

```shell
{
	cp ca.crt /var/lib/kubelet/
	cp node-0.crt /var/lib/kubelet/kubelet.crt
	cp node-0.key /var/lib/kubelet/kubelet.key
}
```

node-1:

```shell
{
  cp ca.crt /var/lib/kubelet/
	cp node-1.crt /var/lib/kubelet/kubelet.crt
	cp node-1.key /var/lib/kubelet/kubelet.key
}
```



Copy the appropriate certificates and private keys to the `server` machine:

```bash
scp \
  ca.key ca.crt \
  kube-api-server.key kube-api-server.crt \
  service-accounts.key service-accounts.crt \
  root@server:~/
```

> The `kube-proxy`, `kube-controller-manager`, `kube-scheduler`, and `kubelet` client certificates will be used to generate client authentication configuration files in the next lab.

Next: [Generating Kubernetes Configuration Files for Authentication](05-kubernetes-configuration-files.md)
