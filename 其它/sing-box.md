# sing-box
sing-box作为一款网络代理平台，高度模块化的设计，对于刚上手的新手配置学习成本较高，但其优点也很明显：
- 协议覆盖全面；
- Apple生态支持最好；
- 性能强劲、TUN 稳定、内存占用小；
- 规则集（Rule Set）机制先进，规则加载迅速。

## 安装
这是[sing-box项目地址](https://github.com/SagerNet/sing-box)，该项目迭代速度很快，配置语法不能向下兼容，对新手不太友好，但熟悉的老鸟即使更新版本也能快速上手。
[下载地址](https://github.com/SagerNet/sing-box/releases#release-v1.14.1)，有各平台二进制安装包。
> [!TIP]
> 本文操作系统平台为`Kylin v10 C86架构桌面版`直接选`Linux amd x86_64`二进制安装包。

- 直接解压安装:
```bash
tar zxvf sing-box-<version>-linux-amd64.tar.gz -C /usr/local/
```
- 启动`sing-box`:
```bash
/usr/local/sing-box/sing-box run -c /usr/local/sing-box/config.json
```

## 配置文件
配置文件是`sing-box`最重要的一部份，文件格式为`json`，文件内顶层每个对象为一个模块，整个配置文件结构：
```json
{
  "$schema": "https://sing-box.sagernet.org/schema.json", // 这个对象模块提供字段补全和校验，比如像Vscode编辑器
  "log": {}, // 配置日志对象
  "dns": {}, // 配置DNS解析模块
  "ntp": {}, // 配置NTP服务同步，比如像vless之类对时间有严格要求的协议
  "certificate": {}, // 配置证书
  "certificate_providers": [], // 证书提供者
  "http_clients": [], // 配置http客户端，比如在路由模块的规则集下载时需要选择哪个http客户端出站
  "network_namespaces": [], // sing-box 跨越不同的 Linux 网络命名空间来运行入站（Inbounds）、出站（Outbounds）、端点（Endpoints）以及 TUN 虚拟网卡
  "endpoints": [], // 端点（Endpoint）是同时具备入站（Inbound）和出站（Outbound）双向行为的协议/网络组件
  "inbounds": [], // 入站
  "outbounds": [], // 出站
  "route": {}, // 路由模块，决定流量规则如何出站
  "services": [], // 让 sing-box 脱离了“单纯的代理内核”，成为一个可以挂载控制面板、自建 VPN 中继（DERP）、网络设备共享（USB/IP）及提供开发辅助服务的通用网络平台
  "experimental": {} // 实验性功能，比如配置缓存开发和缓存路径、配置ui面板。
}
```
> ![!TIP]
> [官方配置文档](https://sing-box.sagernet.org/zh/configuration/)。
> 本文作者可套用的[配置文件示例](/其它/json/sing-box.json)。