# 免费激活安装Offiece

这里所谓的免费激活`Office`主要是指通过`[Office Customization Tool](https://config.office.com/deploymentsettings) 在线生成配置文件`然后再通过`[Office 部署工具](https://www.microsoft.com/en-us/download/details.aspx?id=49117)`进行安装，最后再通过工具来激活`MS Office`。

1. 下载安装`Office Deployment Tool`布署工具。
2. 在`Office Customization Tools`选择配置安装的组件并导出配置文件。


## 开如安装

1. 进入CMD命令行工具，输入以下命令开始下载安装。

```powershell
setup.exe /configure office2024.xml
```

## 完成激活

1. `cd C:\Program Files\Microsoft Office\Office16  或者 cd C:\Program Files(x86)\Microsoft Office\Office16`
2. `cscript ospp.vbs /sethst:kms.loli.best`
3. `cscript ospp.vbs /act`

