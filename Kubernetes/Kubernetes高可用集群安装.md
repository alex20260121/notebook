# Kubernetes高可用集群安装

> [!NOTE]
> 本文安装集群的OS发行版为`Rocky Linux 10.2 x86_64`

集群安装的方式有很多种二进制、`kubeadm`、还有第三方集群安装工具，本文使用官方`kubeadm`安装工具。

## 集群规划

|主机名|主机网络|主机MAC地址|主机UUID|集群平面|
|:-----|:-------|:----------|:-------|:-------|
|Kubernetes-master-node-A1|192.168.122.5|52:54:00:58:65:1e|aba8c4ec-44c7-4655-8a74-c4e8728d0fcb|控制平面|
|Kubernetes-master-node-A2|192.168.122.6|52:54:00:58:65:1e|aba8c4ec-44c7-4655-8a74-c4e8728d0fcb|控制平面|
|Kubernetes-master-node-A3|192.168.122.7|52:54:00:58:65:1e|aba8c4ec-44c7-4655-8a74-c4e8728d0fcb|控制平面|
|Kubernetes-worker-node-A1|192.168.122.8|52:54:00:58:65:1e|aba8c4ec-44c7-4655-8a74-c4e8728d0fcb|工作负载|
|Kubernetes-worker-node-A2|192.168.122.9|52:54:00:58:65:1e|aba8c4ec-44c7-4655-8a74-c4e8728d0fcb|工作负载|
|Kubernetes-worker-node-A3|192.168.122.10|52:54:00:58:65:1e|aba8c4ec-44c7-4655-8a74-c4e8728d0fcb|工作负载|
|loader-blancer-A|192.168.122.11|52:54:00:58:65:1e|aba8c4ec-44c7-4655-8a74-c4e8728d0fcb|负载均衡器|
|loader-blander-B|192.168.122.12|52:54:00:58:65:1e|aba8c4ec-44c7-4655-8a74-c4e8728d0fcb|负载均衡器|

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
默认情况下`Linux`内核不允许数据包在不同网络接口之间转发，得先启用IPv4之间转发:
```zsh
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF
```
- 应用 sysctl 参数而不重新启动:
```zsh
sysctl --system
```

- 验证参数:
```zsh
sysctl net.ipv4.ip_forward
```

#### 1.3.1 安装`containerd`

[下载](https://github.com/containerd/containerd/releases/tag/v2.4.0)二进制安装包，解压安装:
```zsh
tar zxvf containerd-2.4.0-linux-amd64.tar.gz -C /usr/local/
```

#### 1.3.2 `systemd`服务
```ini
# Copyright The containerd Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target dbus.service

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd

Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
```

#### 1.3.3 安装`runc`
[下载](https://github.com/opencontainers/runc/releases)`runc`二进制安装包、解压安装：
```zsh
install -m 755 runc.amd64 /usr/local/sbin/runc
```

#### 1.3.4 安装`CNI`
[下载](https://github.com/containernetworking/plugins/releases)`CNI`二进制安装包、解压安装:
```zsh
mkdir mkdir -pv /opt/cni/bin && tar zxvf cni-plugins-linux-amd64-v1.9.1.tgz -C /opt/cni/bin
```

#### 1.3.5 配置`containerd`
`containerd`默认配置文件路径`/etc/containerd/config.toml`，配置文件语法格式分**`1.x`**版本和**`2.x`**版本，要注意区分否则语法不一样，配置文件不生效。
- 打印默认配置文件:
```zsh
mkdir /etc/containerd && containerd config default > /etc/containerd/config.toml
```
- 将 `runc` 配置为使用 `systemd CGroup` 驱动:
```toml
          [plugins.'io.containerd.cri.v1.runtime'.containerd.runtimes.runc.options]
            BinaryName = ''
            CriuImagePath = ''
            CriuWorkPath = ''
            IoGid = 0
            IoUid = 0
            NoNewKeyring = false
            Root = ''
            ShimCgroup = ''
            SystemdCgroup = true
```