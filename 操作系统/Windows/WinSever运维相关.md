# WinServer 运维
本文用于`WinServer`运维相关的笔记。

## 系统
有关操作系统层面的操作。

### 开机启动时间
```Powershell
systeminfo | findstr "系统启动时间"
systeminfo | findstr "Boot Time"
```
> [!NOTE]
> 这里有个坑，如果是`CMD`终端下敲下这些命令是可以打印系统启动时间。但是在`Powershell`下，在处理 外部命令 | 外部命令 的管道时，默认使用的传递编码（$OutputEncoding）是 US-ASCII，当 systeminfo.exe 输出的中文经过 PowerShell 管道传给 findstr 时，中文会被转换丢失（变成问号 ?），导致 findstr 根本匹配不到 "系统启动时间:"。
改用 PowerShell 原生命令：
```Powershell
systeminfo.exe | Select-String "系统启动时间:"
```

## 磁盘
对于系统内磁盘(硬盘相关的操作)。

## 网络
有关系统内网络栈的操作。

## CPU
操作系统内中央处理器的操作。

## 内存
操作系统内内存相关的操作。