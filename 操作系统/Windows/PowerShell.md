# Windows PowerShell
PowerShell 是一种自动化解决方案，由命令行 shell、脚本语言和配置管理框架组成。

## 命令行
Windows PowerShell 取代了 Windows 命令行接口 (cmd.exe) 及其批处理文件脚本语言的有限功能。 PowerShell 接受并返回 .NET 对象，其中包括：
- 命令行历史记录。
- Tab 自动补全和预测。
- 支持命令和参数别名。
- 链接使用管道功能的命令。
- 可靠的控制台内帮助系统

> [!NOTE]
> `Windows Powershell`只能在`Windows`操作系统内运行，而`Powershell`可以在`MacOs`、`Linux`、`Windows`不同平台运行，`Powershell`使用 .NET Core 实现。

## 脚本语言
`Windows Powershell`脚本简单来说，就是和`Linux`类似，由不同的`Powrshell`命令组成。

## 配置管理框架
PowerShell 中的配置管理框架主要指 DSC（Desired State Configuration，期望状态配置），这是微软官方的声明式配置管理平台。

### 核心概念
声明式 vs 命令式：
- 命令式: 告诉系统怎么做(一步一步执行命令)。
- 声明式（DSC）：告诉系统"要什么状态"，系统自动判断并达成。