# MacOS系统管理
本文整理了关于MacOS系统的管理。
## 管理扩展
在 macOS 中，“系统扩展”（System Extensions）主要分为两类：
1. 现代系统扩展（System Extensions）：macOS 10.15 引入，用于替代内核扩展（如网络扩展、网络过滤、安全防护组件等）。
2. 传统内核扩展（Kernel Extensions / Kext）：位于 /Library/Extensions 的旧版驱动或插件。
### 删除扩展
应用软件一般删除之后扩展也会随之删除，也有一些例外情况，删除应用软件之后因为该应用在后台运行，内存没被释放卡在系统里。

- 查看己安装的软件扩展:
```zsh
systemextensionsctl list
```

- 运行卸载命令（需要填入上面查到的 Team ID 和 Bundle ID）：
```zsh
systemextensionsctl uninstall <TeamID> <BundleID>
```

### 激活扩展
在两个版本同时运行，当前活跃的还在旧版本之上，如果需要使用新版本的扩展，则需要激活。
