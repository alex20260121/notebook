# Kubernetes高可用集群安装

> [!TIP]
> 本文安装集群的OS发行版为`Rocky Linux 10.2 x86_64`

集群安装的方式有很多种二进制、`kubeadm`、还有第三方集群安装工具，本文使用官方`kubeadm`安装工具。

## 集群规划

|主机名|主机网络|主机MAC地址|主机UUID|集群平面|
|:-----|:-------|:----------|:-------|:-------|
|Kubernetes-master-node-A1|192.168.122.5|52:54:00:58:65:1e|aba8c4ec-44c7-4655-8a74-c4e8728d0fcb|控制平面|
|Kubernetes-master-node-A2|192.168.122.6|52:54:00:66:80:03|16fe1979-e27c-4c60-b8b8-e526a9803896|控制平面|
|Kubernetes-master-node-A3|192.168.122.7|52:54:00:B7:3A:B0|99e360b9-68d6-454e-93e3-ea7a82078cf5|控制平面|
|Kubernetes-worker-node-A1|192.168.122.8|52:54:00:bc:02:33|de8d9222-ba57-40af-b194-71503e4c763d|工作负载|
|Kubernetes-worker-node-A2|192.168.122.9|52:54:00:fa:2a:e0|bd1e6185-3503-4e8e-a050-e27235da1253|工作负载|
|Kubernetes-worker-node-A3|192.168.122.10|52:54:00:03:a6:b7|f5199ee2-4bf8-4e42-9663-fb35eadbcdd7|工作负载|
|loader-blancer-A|192.168.122.11|52:54:00:f4:f0:49|0ee442fc-52fe-42f1-bcc4-693e6ede0735|负载均衡器|
|loader-blander-B|192.168.122.12|52:54:00:d3:13:04|34f71325-bfa5-4c9b-ba61-d35e3ab2416c|负载均衡器|

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

### 1.5 安装容器运行时
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

#### 1.5.1 安装`containerd`

[下载](https://github.com/containerd/containerd/releases/tag/v2.4.0)二进制安装包，解压安装:
```zsh
tar zxvf containerd-2.4.0-linux-amd64.tar.gz -C /usr/local/
```

#### 1.5.2 `systemd`服务
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

#### 1.5.3 安装`runc`
[下载](https://github.com/opencontainers/runc/releases)`runc`二进制安装包、解压安装：
```zsh
install -m 755 runc.amd64 /usr/local/sbin/runc
```

#### 1.5.4 安装`CNI`
[下载](https://github.com/containernetworking/plugins/releases)`CNI`二进制安装包、解压安装:
```zsh
mkdir mkdir -pv /opt/cni/bin && tar zxvf cni-plugins-linux-amd64-v1.9.1.tgz -C /opt/cni/bin
```

#### 1.5.5 配置`containerd`
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

#### 1.5.6 配置pause镜像
配置特定版本的沙箱(pause)镜像:
```toml
[plugins.'io.containerd.cri.v1.images'.pinned_images]
  sandbox = 'registry.k8s.io/pause:3.10.2'
```

- 重载`systemd`:
```zsh
systemctl daemon-reload
```

- 设置为开机启动:
```zsh
systemctl enable containerd.service --now
```

#### 1.5.7 安装`kubeadm`、`kubectl`、`kubelet`
- 关闭`SElinux`:
```zsh
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
```
> [!TIP]
> 修改完之`selinux`配置文件后需要重启生效`reboot`

- 添加 Kubernetes 的 yum 仓库:
```zsh
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://pkgs.k8s.io/core:/stable:/v1.37/rpm/
enabled=1
gpgcheck=1
gpgkey=https://pkgs.k8s.io/core:/stable:/v1.37/rpm/repodata/repomd.xml.key
exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
EOF
```
> [!TIP]
> `exclude`包含的值，在`yum`或`dnf`升级时会被锁定，不会跟随升级。

- 安装:
```zsh
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
```
> [!TIP]
> 这里的`--disableexcludes=kubernetes`表示临时关闭`kubernetes`软件源的`exclude`规则。

- 设置`kubelet`开机启动:
```zsh
systemctl enable kubelet.service --now
```
> [!TIP]
> 在集群没安装好之前`kubelet`会进入一个循环重启过程，直到集群准备就绪。

## 2. 安装时间同步服务
Kubernetes 本质上是一个高度复杂的分布式系统，节点之间的时间哪怕只相差几秒甚至几百毫秒，都可能导致核心组件崩溃、认证失败或数据混乱。

### 2.1 安装`chronyd`
这里`chronyd`时间同步服务只需要一台连接公网同步时间，基它所有节点的时间同步服务地址都指向这台向外网同步时间的机器。
> [!TIP]
> 为了更好的区分集群内时间同步主机的角色，这里直接称向外网同步时间的主机为"主节点"，其它向这台"主节点"同步的机器为"从节点"。

```zsh
dnf -y install chronyd
```

### 2.2 配置文件
- 主节点服务配置文件:
```ini
# 包含阿里云时间同步域名，最小轮询间隔和最大轮询间隔。
server ntp.aliyun.com minpoll 4 maxpoll 10 iburst
# 允许时间同步的客户端
allow 192.168.122.0/24
```
- 从节点服务配置文件:
```ini
# 指向主节点的`chronyd`服务
server 192.168.122.5 iburst
```
- 重启主从`chronyd`服务:
```zsh
systemctl restart chronyd.service
```
- 查看时钟源同步状态：
```zsh
chronyc sources

# 如下，*代表选中使用该时钟源。
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* 203.107.6.88                  2   4   377    11  -3551us[-5679us] +/-   35ms
```

## 3. 高可用及负载均衡器
这里的高可用负载均衡指的是对`Kubernetes`集群的控制平面组件`(api-server)`和`etcd`，这两个控制平面的组件，这两组件可以单独拆开布署，本文档控制平面的组件全部放在一起。

- `nginx`作为整个`api-server`前端流量入口的负载均衡器;
- `KeepAlived`为`nginx`负载均衡器提供一个由可配置的健康检查机制管理的`VIP`，当一台主机网络不可用时，`VIP`会飘移到另一台主机上继续提供服务。

### 3.1 安装组件
因为`nginx`要用到反向代理，所以要安装`upstream`模块其它依赖都会自动安装。
```zsh
dnf -y install nginx-mod-stream-2:1.26.3-6.el10_2.6.x86_64 keepalived-2.2.8-9.el10.x86_64
```

### 3.2 配置`keepalived`
```ini
! /etc/keepalived/keepalived.conf
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
}
vrrp_script check_apiserver {
  script "/etc/keepalived/check_apiserver.sh"
  interval 3
  weight -2
  fall 10
  rise 2
}

vrrp_instance VI_1 {
    state 
    interface ens3
    virtual_router_id 51
    priority 100
    authentication {
        auth_type PASS
        auth_pass 42
    }
    virtual_ipaddress {
        192.168.122.100
    }
    track_script {
        check_apiserver
    }
}
```

### 3.3 `KeepAlived`健康检查脚本
在`KeepAlied`配置文件定义好的路径下编辑`/etc/keepalived/check_apiserver.sh`:

```bash
#!/bin/sh

errorExit() {
    echo "*** $*" 1>&2
    exit 1
}
APISERVER_DEST_PORT=16443

curl -sfk --max-time 2 https://localhost:${APISERVER_DEST_PORT}/healthz -o /dev/null || errorExit "Error GET https://localhost:${APISERVER_DEST_PORT}/healthz"
```
添加脚本执行权限`chmod 777 /etc/keepalived/check_apiserver.sh`

### 3.4 配置`Nginx`
使用 upstream 块定义后端 API 服务器集群，并在 server 块中配置反向代理转发请求。配置文件路径: `/etc/nginx/conf.d/k8s-apiserver.conf`。
```ini
# 后端 API 服务器集群
upstream k8s-apiserver{
  server 192.168.122.5:6443 weight=3 max_fails=3 fail_timeout=10s;
  server 192.168.122.6:6443 weight=2 max_fails=3 fail_timeout=10s;
  
  # 备用服务器(当其它服务器全部不可用时)
  server 192.168.122.7:6443 backup;
}

server {
  listen 16443;

  # 客户端最大请求体大小限制10M
  client_max_body_size 10M;

  location / {
    proxy_pass http://k8s-apiserver;

    # 传递真实客户端IP及协议头
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    # 超时设置
    proxy_connect_timeout 5s;
    proxy_read_timeout 60s;
    proxy_send_timeout 60s;

    # 启用 K8s 专属日志文件，并使用上面定义的格式
    access_log /var/log/nginx/k8s_apiserver_access.log main;
    error_log /var/log/nginx/k8s_apiserver_error.log warn;
  }
}
```

### 3.5 配置开机启动
```bash
systemctl enable keepalived.service --now && systemctl enable nginx.service --now
```

## 4. 创建集群
使用`kubeadm init`初始化创建一个新集群，在这过程中`kubeadm`工具会自动去拉取集群所需要的容器镜像，集群前置准备检测工作，自动掰发TLS证书...等等。
