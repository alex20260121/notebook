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
