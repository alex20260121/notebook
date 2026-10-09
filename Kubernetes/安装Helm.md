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

