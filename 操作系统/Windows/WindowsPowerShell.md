# WindowsPowerShell
`windows Powershell` 是一种类似于 `Linux shell` 的命令行工具，用于自动执行基于 `Windows` 的系统的管理任务。 `Windows PowerShell` 预安装在 `Windows` 操作系统的所有新式版本上。

## PowerShell cmdlet 的结构
PowerShell cmdlet 使用的常见格式是“动词-名词”表示法。

### Cmdlet 动词
cmdlet 名称的谓词部分指示该 cmdlet 执行的操作。常见谓词包括：
- Get: 检索资源，例如文件或用户。
- Set: 更改与资源相关的数据，例如文件或用户属性。
- New: 创建资源，例如文件或用户。
- Add: 将资源添加到多个资源的容器。
- Remove: 将资源从多个资源的容器中删除。

> [!TIP]
> 可以运行 Get-Verb 该命令，获取谓词的完整列表。

### Cmdlet 名词
cmdlet 名称的名词部分指示该 cmdlet 影响的资源或对象的类型。对同一资源运行的所有 cmdlet 应使用相同的名词。 例如，Service 名词适用于使用 Windows 服务的 cmdlet，而 Process 名词用于管理计算机上的进程。

名词中还可以包含前缀，可帮助将相关名词按系列分组。 例如，Active Directory名词以字母 AD开头（如 ADUser、ADGroup 和 ADComputer）。 Microsoft SharePoint 服务器 cmdlet 以前缀 SP 开头，Microsoft Azure cmdlet 以前缀 Az 开头。

### 参数
参数名称以短划线 (-) 开头。 空格用于分隔要从参数名称传递的值。 如果传递的值包含空格，需要用引号将文本引起来。 某些参数接受多个值，这些值必须用逗号分隔（不含空格）。

#### 可选参数与必需参数
参数可以为可选参数或必需参数。 如果是必需参数，而运行 cmdlet 时没有提供该参数的值，Windows PowerShell 会提示你为其提供值。

在某些情况下，可选择不输入参数名称，只需输入参数的值即可。 运行命令 Get-ChildItem C:\ 与运行命令 Get-ChildItem -Path C:\ 相同，因为参数 -Path 在 cmdlet 定义中定义为第一个参数。 这称为位置参数。 

#### 布尔值
某些参数接受类似`true`或`false`的布尔值，与实际布尔值不同的是，当命令包含了该开关参数，才会相当于显示指定了`True`,否则就是`False`，一个示例是 Get-ChildItem cmdlet 的 -Recurse 参数或开关。 Get-ChildItem c:\ -Recurse 命令不仅会返回 C:\ 目录中的项，还会返回其所有子目录中的项。 如果没有 -Recurse 开关，将仅返回 C:\ 目录中的项。

### Tab 补全功能
Windows Powershell有类似于Linux的Tab补全命令行功能，只需要给出模糊前几个字符按下Tab键就可以显示更多可用选项，如果知道需要适用于Active Directory资源，则可以在控制台中输入文本 `Get-AD`，然后按 Tab 键查看可用选项。
对于参数，只需输入短划线 (-)，然后可以多次按 Tab 键查看 cmdlet 的所有参数。

Tab 自动补全甚至可以使用通配符。 如果你知道某个 cmdlet 在服务上运行，但不确定需要哪个 cmdlet，请在控制台中输入文本 *-service ，然后按 Tab 键查看名称中包含文本 -service 的所有 cmdlet。

### 帮助主题
虽然 Windows PowerShell 中的很多帮助内容与命令有关，但也有很多帮助文件描述了 PowerShell 概念。 这些文件包括有关 PowerShell 脚本语言、运算符和其他详细信息的信息。 此信息并不具体涉及单个命令，而是涉及全局 shell 技术和功能。

- 可以通过运行以下命令来查看这些主题的完整列表：
```Powershell
Get-Help about*
```

- 查看指定主题：
```Powershell
Get-Help about_CommonParameters
```

- 还可以在单独的窗口中或联机打开帮助主题：
```Powershell
Get-Help about_CommonParameters -ShowWindow
Get-Help about_CommonParameters -Online
```

