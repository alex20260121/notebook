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
