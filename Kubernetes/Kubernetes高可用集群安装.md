# Kubernetes高可用集群安装

> [!NOTE]
> 本文安装集群的OS发行版为`Rocky Linux 10.2 x86_64`

集群安装的方式有很多种二进制、`kubeadm`、还有第三方集群安装工具，本文使用官方`kubeadm`安装工具。

## 集群规划

|主机名|主机网络|主机MAC地址|主机UUID|集群平面|
|:-----|:-------|:----------|:-------|:-------|
|Kubernetes-master-node-A1|192.168.122.5|52:54:00:58:65:1e|aba8c4ec-44c7-4655-8a74-c4e8728d0fcb|控制平面|


## 1. 安装`kubeadm`

安装`Kubernetes`集群需要满足一定的条件：
- 兼容`Kubernetes`指令的`Linux`操作系统(Debian、Red Hat);
- 至少2GB的内存大小;
- 至少2Core的CPU规格;
- 节点之中不可以有重复的主机名、MAC 地址或 product_uuid;
- 集群中的主机网络可用性必须能相互连接;
- 集群中主机的端口必须在防火墙端放行;

### 1.1 集群端口

- 控制平面:

|协议|方向|端口|服务|使用者|
|:---|:---|:---|:---|:-----|
|TCP|入站|6443|api-server|所有|
|TCP|入站|2379-2380|etcd 服务器客户端 API|kube-apiserver、etcd|
|TCP|入站|10250|kubelet API|自身、控制面|
|TCP|入站|10259|kube-scheduler|自身|
|TCP|入站|10257|kube-controller-manager|自身|

- 负载节点:

|协议|方向|端口|服务|使用者|
|:---|:---|:---|:---|:-----|
|TCP|入站|10250|kubelet API|自身、控制面|
|TCP|入站|10256|kube-proxy|自身、负载均衡器|
|TCP|入站|30000-32767|NodePort Services†|所有|
|UDP|入站|30000-32767|NodePort Services†|所有|

### 1.2 查看MAC地址唯一性

```zsh
ip link
```

### 1.3 查看主机`product_id`唯一性

```zsh
cat /sys/class/dmi/id/product_uuid
```
### 1.4 关闭交换分区

```zsh
swapoff -a
```
> [!TIP]
> 或者直接在`/etc/fstab`开机自动挂载表下注释掉`SWAP`交换分区。

### 1.3 安装容器运行时
