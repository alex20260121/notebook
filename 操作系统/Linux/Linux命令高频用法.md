# Linux命令排障高频用法

当遇到故障或系统性能瓶颈需要分析时，一条命令下多个参数和子参数如何组合成高效直观的排查方法？
## 网络

以下为`Linux`操作系统下网络命令的高频用法。

### ss

ss（Socket Statistics）是 Linux 下用来替代传统 netstat 的现代网络排查利器。它直接通过内核的 Netlink 接口（tcp_diag 模块）获取套接字信息，在服务器连接数达到数万甚至数十万时，netstat 可能会卡死，而 ss 依然能毫秒级返回结果。

要用好 ss，核心在于：记住高频参数组合 + 掌握内置过滤语法 + 学会看懂关键指标。

#### 核心参数

- 协议类型：`-t (TCP)、-u (UDP)、-x (Unix Domain Socket)、-w (RAW)`

- 状态筛选：-l (Listening，仅看监听端口)、-a (All，看所有连接，包括监听和已建立)

- 提速与防坑：-n (Numeric，强烈建议必加，不解析域名和端口服务名，直接显示 IP 和数字端口，速度极快)

- 溯源定位：-p (Process，显示占用该连接的进程名和 PID，通常需要加 sudo 才能看到所有进程)

- 深度排查：

    - `-s`: 汇总统计当前各类连接的数量。
    - `-i`: 显示 TCP 内部信息（如拥塞控制算法、RTT 延迟、重传、窗口大小等）。
    - `-e`：显示扩展信息（如 UID、inode、cookie 等）。
    - `-m`：显示套接字占用的内存情况（排障内存泄漏或缓存区瓶颈时很有用）。
    - `-o`：显示 TCP 定时器信息（如 keepalive、retrans 等）。

#### 运维高频用法

> [!TIP]
> 在没有`root`权限下需要加上`sudo`，普通用户是无法看到`root`开启的进程。

1. 查看本机开放了哪些端口，被谁占用了。
```bash
ss -tulnp
```

2. 快速评估服务器当前网络负载（连接数概览），输出一个简洁的汇总表，能立刻看出总连接数、estab（已建立）、closed、orphaned、timewait 各自有多少，比用 wc -l 统计快得多。
```zsh
ss -s
```

3. 杳看所有的TCP连接, 查看所有（a）TCP（t）连接的数字地址（n）和对应进程（p）。如果只想看 IPv4 或 IPv6，可额外加上 -4 或 -6。
```zsh
ss -tanp
```

4.  统计各种 TCP 状态的分布（配合 awk）
```zsh
ss -tan | awk 'NR>1 {print $1}' | sort | uniq -c | sort -nr
```
> [!TIP]
> 如果发现大量 CLOSE_WAIT，通常是应用程序代码里没有正确关闭 socket 导致的；如果大量 SYN_RECV，可能是遭受了 SYN Flood 攻击或半连接队列溢出。

#### 进阶用法(过滤)
很多人习惯用 ss -tan | grep :80，但在连接数极大时，先输出再 grep 效率低且容易误匹配（比如匹配到其他 IP 里的 :80）。ss 自带了类似 tcpdump 的高效过滤表达式（在内核层过滤），不用 grep 就能精准定位。

1. 按连接状态过滤（state），常见状态关键字：`established、listening、time-wait、close-wait、syn-recv、fin-wait-1、fin-wait-2`。
```zsh
# 只看处于 ESTABLISHED 状态的 TCP 连接
ss -tan state established

# 只看处于 TIME-WAIT 状态的连接
ss -tan state time-wait

# 组合使用：查看所有已经建立连接或相关进程
sudo ss -tanp state established
```

2. 按 IP 和端口过滤（`src / dst / sport / dport`），端口号前面必须加冒号 :，比较操作符和括号两侧建议留空格，表达式用单引号括起来防止被 Shell 解析。
```zsh
# 查看目标端口是 443 的连接（作为客户端去连别人）
ss -tan dport = :443

# 查看源端口是 80 的连接（作为服务端提供 80 服务）
ss -tan sport = :80

# 简化写法（用 dst / src 同时匹配 IP 和端口）：
# 查看与 192.168.1.50 这台机器的所有连接
ss -tan dst 192.168.1.50

# 查看来自 10.0.0.0/8 网段的连接
ss -tan src 10.0.0.0/8

# 复杂逻辑组合（or / and / not）：
# 查找本地或对端端口为 80 或 443 的已建立连接
ss -tan state established '( dport = :80 or sport = :80 or dport = :443 or sport = :443 )'

# 端口范围比较（大于 1024 的端口：gt/ge/lt/le/eq/ne）
ss -tan sport gt :1024
```

#### 用 ss 诊断网络性能与异常

1.  排查网络卡顿、丢包与拥塞（-i 与 -m），当业务反馈“连接能通但非常慢”时，用 -ti 查看内核级 TCP 指标：
```zsh
ss -tin dst 192.168.1.50
```
重点关注输出中的这几个字段：

- `Recv-Q / Send-Q`：
  - 对于 LISTEN 状态的套接字：Recv-Q 表示当前全连接队列（Accept Queue）中等待应用程序 accept() 的连接数，Send-Q 表示该队列的最大容量（somaxconn）。如果 Recv-Q 持续接近或等于 Send-Q，说明应用程序处理太慢，正在丢弃新连接！
  - 对于 ESTAB 状态的套接字：Recv-Q 是内核收到但应用还没读走的数据字节数；Send-Q 是本机发出但对方还没 ACK 确认的字节数。这两个值长期不为 0，通常意味着应用读取卡顿或网络拥塞。
- rtt:12.5/3.2：往返延迟均值及方差（单位毫秒），能直观判断链路是否抖动。
- cwnd:10：拥塞窗口大小。如果持续很小，说明网络存在拥塞或刚经历过重传。
retrans:0/5：当前正在重传的包数 / 历史总重传数。如果总重传数持续攀升，说明链路丢包严重。
- cubic / bbr：当前使用的 TCP 拥塞控制算法。
- 加上 -m（ss -tinm）还能看到 skmem:(r0,rb87380,t0,tb16384...)，其中 r/t 是当前读写缓冲区的内存占用，rb/tb 是读写缓冲区的上限，d 表示因缓冲区满而丢弃的包数（如果 d 大于 0，就需要考虑调大 rmem 缓冲区了）。

## 磁盘