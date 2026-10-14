# Helm概述
`Helm`是`Kubernetes`的包管理器，类似`ubuntu`的`apt`或`RedHat`的`dnf, yum`包管理器，`Helm`可以对`Kubernetes`集群中运行的应用程序进行管理，如安装、升级、回滚。

## 安装
`Helm`提供两种安装方法：
1. 发布的二进制安装;
2. 操作系统的包管理器安装;

### 二进制安装
[官方项目](https://github.com/helm/helm/releases)二进制发布页面下载地址，选择版本下载：
```zsh
wget https://get.helm.sh/helm-v4.3.0-linux-amd64.tar.gz
```
- 解压
```zsh
tar zxvf helm-v4.3.0-linux-amd64.tar.gz
```

- 将可执行二进制文件移动至目标目录
```zsh
sudo mv linux-amd64/helm /usr/local/bin/
```

### 通过`Service`暴露端口
- 只在本机环回端口监听
```bash
kubectl port-forward -n kube-system service/headlamp 8080:80
```

- 监听任意网卡
```bash
kubectl port-forward --address 0.0.0.0 -n kube-system services/my-headlamp 8088:80
```

### 登陆`DashBoard`
创建一个`Headlamp`的服务账号通过令牌认证。
1. 创建服务账号(sa):
```bash
kubectl -n kube-system create serviceaccount headlamp-admin
```
2. 赋于管理员权限
```bash
kubectl create clusterrolebinding headlamp-admin --serviceaccount=kube-system:headlamp-admin --clusterrole=cluster-admin
```
3. 创建服务账号的令牌
```bash
kubectl create token headlamp-admin -n kube-system
```
