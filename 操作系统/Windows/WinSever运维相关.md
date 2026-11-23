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

### PID查看进程名
- Powershell
```Powershell
Get-Process -Id 784

# 只要进程名
(Get-Process -Id 1234).ProcessName
```

- tasklist（最通用，CMD/PowerShell 都行）
```CMD
tasklist /FI "PID eq 1234"
```

- 反查：知道了 PID 想看完整命令行
```Powershell
Get-CimInstance Win32_Process -Filter "ProcessId=1234" | Select-Object Name,ProcessId,CommandLine
```

## 磁盘
对于系统内磁盘(硬盘相关的操作)。

## 网络
有关系统内网络栈的操作。

### 查看当前可用的动态端口范围
```Powershell
netsh int ipv4 show dynamicport tcp
netsh int ipv4 show dynamicport udp
netsh int ipv6 show dynamicport tcp
```
> [!NOTE]
> 也就是动态端口范围通常是 49152–65535（共 16384 个）。

- 用 PowerShell 找出动态范围内真正空闲（未监听）的端口
```Powershell
# 查看当前被监听/占用的端口
Get-NetTCPConnection | Select-Object LocalPort,State,OwningProcess | Sort-Object LocalPort
```
或列出某个范围内未被占用的端口:
```Powershell
$used = (Get-NetTCPConnection -State Listen).LocalPort
49152..65535 | Where-Object { $_ -notin $used }
```

- 修改动态端口范围（如需固定，管理员权限）:
```Powershell
netsh int ipv4 set dynamicport tcp start=49152 num=16384
```

### 查看网络连接状态
```Powershell
# 查看己建立的连接
netstat -ano | findstr "ESTABLISHED" 或 netstat -ano | Select-String "ESTABLISHED"

# 查看正处于监听的端口
netstat -ano | Select-String "LISTENING"

# 查看正等待关闭的连接
netstat -ano | Select-String "CLOSE_WAIT"
```

## CPU
操作系统内中央处理器的操作。

## 内存
操作系统内内存相关的操作。