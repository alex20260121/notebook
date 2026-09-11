# kvm 虚拟化平台
- KVM（Kernel-based Virtual Machine，基于内核的虚拟机），KVM是Linux操作系统的内核模块，负责虚拟化`CPU`和`Memory`，它依赖`kvm.ko`及特定架构的`kvm_intel.ko / kvm_amd.ko`模块。
- QEMU（外设与设备的管家），QEMU 是一个运行在用户空间的程序。如果只有 CPU 和内存，虚拟机根本无法启动，因此 QEMU 负责模拟虚拟机的全部周边硬件（BIOS/UEFI 固件、PCI 总线、IDE/SATA 硬盘控制器、Realtek/Intel 网卡、显示卡等）。
- Virtio（半虚拟化驱动） 应运而生：虚拟机操作系统如果安装了 Virtio 驱动（例如 virtio-net、virtio-blk），虚拟机就“知道”自己运行在虚拟环境中，它不再傻傻地模拟硬件中断，而是通过宿主机与客户机共享的一块内存队列（Virtqueue）直接高效地传送数据，让 I/O 性能逼近物理硬件。

## 安装
现发行版的`Linux`操作系统一般都集成了`kvm`模块，一般执行`lsmod|grep kvm`就会输出类似`kvm_amd`或`kvm_intel`相关字符。还有需要查看一下BIOS是否开启了虚拟化的支持`egrep -c '(vmx|svm)' /proc/cpuinfo`，一般有多少核心回显对应的数字。

```bash
sudo apt-get -y install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager qemu-img
```

- 将用户添加到`libvirt`和`kvm`系统组，这样不需要`root`权限就可以管理虚拟化平台。
```bash
sudo usermod -aG libvirt $USER
sudo usermod -aG kvm $USER
```
## 创建虚拟机
使用`VNC`图形连接安装。

```zsh
virt-install --name Rocky-10.2-x86_64-minimal \                                                                                
--vcpu 2 \
--memory 4096 \
--disk virtualization/disks/Rocky-10.2-x86_64-minimal.qcow2,size=20,format=qcow2 \
--network network=default \
--location virtualization/images/Rocky-10.2-x86_64-minimal.iso \
--graphics vnc,port=5901,listen=0.0.0.0 \
--noautoconsole
```
## 配置`console`控制台
> [!NOTE]
> 本文档使用的`Linux`发行版均使用`Rocky Linux 10.2`。

在 Rocky Linux 中，最标准、最直接的配置方式是使用 grubby 工具。请在虚拟机内按以下步骤操作：

### 1. 使用 grubby 调整内核参数
在 BLS 机制下，每个内核的启动参数都独立存放在 /boot/loader/entries/ 目录下的 .conf 文件中。直接编辑 /etc/default/grub 并不会自动修改已有的内核启动参数，且 Rocky 默认带有 rhgb quiet，导致关机、重启过程完全静默。
- 移除静默启动参数（删掉 rhgb 和 quiet，解除日志屏蔽）：
```zsh
grubby --update-kernel=ALL --remove-args="rhgb quiet"
```
- 添加串口控制台输出（同时输出到 VNC 的 tty0 和串口的 ttyS0）：
```zsh
grubby --update-kernel=ALL --args="console=tty0 console=ttyS0,115200n8"
```
- 查看并确认修改是否成功写入：
```zsh
sudo grubby --info=DEFAULT
```
> [!TIP]
> 检查输出中的 args= 这一行：
> - 确认没有 rhgb 和 quiet。
> - 确认包含 console=tty0 console=ttyS0,115200n8。

### 2. 确保串口登录服务开机自启
确认终端服务已设为开机自启：
```zsh
systemctl enable serial-getty@ttyS0.service --now
```

### 3. （可选）：让 GRUB 倒计时选单也显示在串口上
