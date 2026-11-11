# OpenClaw

之前`OpenClaw`刚出现的时候玩过一两次，只是简单的安装、运行、然后接上`DeepSeek`的`API KEY`进行日常重复性工作的替代。如何真正玩好这个智能体并不太熟悉。

本文整理了一下`OpenClaw`相关的运维的操作。
## 配置文件

`OpenClaw`的配置文件在`$HOME/.openclaw/openclaw.json`其中大部份使用的是默认值，需要定制化的话还是得看明白配置文件内容。

### 网关

网关是`OpenClaw`所有请求来源的入口，包括`session`、`router`、`chanel`等。

