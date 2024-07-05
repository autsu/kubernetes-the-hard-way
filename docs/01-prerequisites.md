# Prerequisites

In this lab you will review the machine requirements necessary to follow this tutorial.

## Virtual or Physical Machines

This tutorial requires four (4) virtual or physical ARM64 machines running Debian 12 (bookworm). The follow table list the four machines and thier CPU, memory, and storage requirements.

准备以下 4 台机器，每台机器的用途及配置需求如表格所示。

（jumpbox 这台的话，主要就是下载 k8s 组件，以及一些相关的配置文件，并传输给 k8s master 和 k8s service，所以用你自己的电脑就可以，无需专门搭建一台虚拟机作为 jumpbox。）

注意：k8s 节点（也就是 server, node-0, node-1 这 3 个）的主机名非常重要，你最好直接按照下表的名称进行命名，如果你要自定义命名的话，需要修改后续章节中生成节点相关证书的命令，保证和你的主机名相符（没必要折腾自己）。否则，会在工作节点 kubelet 注册到 master 时报错。

我一开始就给这三个节点重新命了个名：master1, worker1, worker2，但是生成证书我又是用的 server, node-0, node-1，这导致 kubelet 报错：

Unable to register node with API server" err="nodes \"worker1\" is forbidden: node \"node-0\" is not allowed to modify node \"worker1\

修改主机名为 node-0 然后 reboot 后解决了问题，所以，主机名非常重要。

| Name    | Description            | CPU | RAM   | Storage |
|---------|------------------------|-----|-------|---------|
| jumpbox | Administration host    | 1   | 512MB | 10GB    |
| server  | Kubernetes server      | 1   | 2GB   | 20GB    |
| node-0  | Kubernetes worker node | 1   | 2GB   | 20GB    |
| node-1  | Kubernetes worker node | 1   | 2GB   | 20GB    |

How you provision the machines is up to you, the only requirement is that each machine meet the above system requirements including the machine specs and OS version. Once you have all four machine provisioned, verify the system requirements by running the `uname` command on each machine:

```bash 
uname -mov
```

After running the `uname` command you should see the following output:

```text
#1 SMP Debian 6.1.55-1 (2023-09-29) aarch64 GNU/Linux
```

You maybe surprised to see `aarch64` here, but that is the official name for the Arm Architecture 64-bit instruction set. You will often see `arm64` used by Apple, and the maintainers of the Linux kernel, when referring to support for `aarch64`. This tutorial will use `arm64` consistently throughout to avoid confusion.

Next: [setting-up-the-jumpbox](02-jumpbox.md)
