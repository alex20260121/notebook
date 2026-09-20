# Kubernetes高可用集群安装
集群安装的方式有很多种二进制、`kubeadm`、还有第三方集群安装工具，本文使用官方`kubeadm`安装工具。
## 1. 安装前置条件
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