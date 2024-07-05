# Provisioning Pod Network Routes

Pods scheduled to a node receive an IP address from the node's Pod CIDR range. At this point pods can not communicate with other pods running on different nodes due to missing network [routes](https://cloud.google.com/compute/docs/vpc/routes).

In this lab you will create a route for each worker node that maps the node's Pod CIDR range to the node's internal IP address.

> There are [other ways](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-achieve-this) to implement the Kubernetes networking model.



调度到节点的 Pod 接收来自该节点的 Pod CIDR 范围的 IP 地址。此时，由于缺少网络路由，Pod 无法与不同节点上运行的其他 Pod 进行通信。

在本实验中，您将为每个工作节点创建一条路由，将节点的 Pod CIDR 范围映射到节点的内部 IP 地址。

还有其他方法可以实现 Kubernetes 网络模型。

## The Routing Table

In this section you will gather the information required to create routes in the `kubernetes-the-hard-way` VPC network.

Print the internal IP address and Pod CIDR range for each worker instance:

```bash
{
  SERVER_IP=$(grep server machines.txt | cut -d " " -f 1)
  NODE_0_IP=$(grep node-0 machines.txt | cut -d " " -f 1)
  NODE_0_SUBNET=$(grep node-0 machines.txt | cut -d " " -f 4)
  NODE_1_IP=$(grep node-1 machines.txt | cut -d " " -f 1)
  NODE_1_SUBNET=$(grep node-1 machines.txt | cut -d " " -f 4)
}
```

```bash
ssh root@server <<EOF
  ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
  ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
EOF

# me
echo "ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}"
echo "ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}"

or

echo "ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}" | envsubst

# 输出
ip route add 10.200.0.0/24 via 198.19.249.34
ip route add 10.200.1.0/24 via 198.19.249.136

然后手动执行（没办法，不能 ssh 机器） 
该命令的作用：系统会知道，任何试图到达 10.200.0.0/24 网络的数据包应该通过网关 198.19.249.34。
```

```bash
ssh root@node-0 <<EOF
  ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
EOF

# me

echo ip route add ${NODE_1_SUBNET} via ${NODE_1_IP}
ip route add 10.200.1.0/24 via 198.19.249.136

在 node-0 上手动执行 ip route add 10.200.1.0/24 via 198.19.249.136
```

```bash
ssh root@node-1 <<EOF
  ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
EOF

# me

echo ip route add ${NODE_0_SUBNET} via ${NODE_0_IP}
ip route add 10.200.0.0/24 via 198.19.249.34

在 node-1 上手动执行 ip route add 10.200.0.0/24 via 198.19.249.34
```

## Verification 

```bash
ssh root@server ip route
```

```text
default via XXX.XXX.XXX.XXX dev ens160 
10.200.0.0/24 via XXX.XXX.XXX.XXX dev ens160 
10.200.1.0/24 via XXX.XXX.XXX.XXX dev ens160 
XXX.XXX.XXX.0/24 dev ens160 proto kernel scope link src XXX.XXX.XXX.XXX 
```

![image.png](https://raw.githubusercontent.com/autsu/diagrams/master/img/20240706004707.png)



```bash
ssh root@node-0 ip route
```

```text
default via XXX.XXX.XXX.XXX dev ens160 
10.200.1.0/24 via XXX.XXX.XXX.XXX dev ens160 
XXX.XXX.XXX.0/24 dev ens160 proto kernel scope link src XXX.XXX.XXX.XXX 
```



![image.png](https://raw.githubusercontent.com/autsu/diagrams/master/img/20240706004737.png)



```bash
ssh root@node-1 ip route
```

```text
default via XXX.XXX.XXX.XXX dev ens160 
10.200.0.0/24 via XXX.XXX.XXX.XXX dev ens160 
XXX.XXX.XXX.0/24 dev ens160 proto kernel scope link src XXX.XXX.XXX.XXX 
```

![image.png](https://raw.githubusercontent.com/autsu/diagrams/master/img/20240706004807.png)



Next: [Smoke Test](12-smoke-test.md)
