# 源码编译安装tmux
其实完全没必要手动源码编译安装`tmux`，直接一条命令可以安装`brew install tmux`，但是就是喜欢折腾。

> [!TIP]
> [Homebrew](https://brew.sh/)这个是前置条件，必须得先安装。

## 安装 Xcode 命令行工具
一般买Mac电脑的不是设计人员就是开发人员，如果是开发人员`xcode`安装是开机之后第一件要做的事情。
```zsh
xcode-select --install
```

### 安装构建依赖项
编译`tmux`需要`libeventand ncurses`
```zsh
brew install automake pkg-config libevent ncurses
```

### 克隆`git`仓库
```zsh
git clone https://github.com/tmux/tmux.git
```

### 生成编译配置文件(configure)
```zsh
cd tmux && sh autogen.sh
```

### 配置、编译、安装
```zsh
# 配置、编译
./configure --disable-ble-utf8proc --disable-jemalloc && make

# 安装
sudo make install
```
> [!TIP]
> 安装完毕查看一下版本`tmux -V`

## MacOS最佳体验
`MacOS`的特定配置和性能调优，可以获得`tmux`的最佳体验。

### 原生剪帖板集群
macOS 使用 ` pbcopyv1` 和pbpaste`v2` 进行剪贴板访问。添加以下内容~/.tmux.conf即可启用 vi 快捷键绑定的复制模式剪贴板支持。
```zsh
bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "pbcopy"
bind -T copy-mode-vi MouseDragEnd1Pane send-keys -X copy-pipe-and-cancel "pbcopy"
```
### Item2控制模式
iTerm2 通过控制模式（`flag`）原生集成了 tmux -CC。这会将 tmux 窗口和窗格直接映射到 macOS 原生的 iTerm2 标签页和分屏窗口。
```zsh
tmux -CC
tmux -CC attach
```