# Oh-My-ZSH终端美化
> [!NOTE]
> 以下操作均在`Kylin v10`桌面版下执行，OS平台不同需要根据不同平台环境下操作。

Oh My Zsh 是一个开源的终端美化神器，它不会让你成会一个10倍效率的开发者，但是其内置/社区主题和丰富的插件，可以自定义和美化你的终端。

## 1. 安装`ZSH`
安装Oh My Zsh之前需要安装`ZSH`，运行`zsh --version`检查是否己安装，如果回显提示`command not found`就需要安装`zsh`了。
- 安装`ZSH`:
```zsh
sudo apt-get -y install zsh
```
- 验证安装，兼容`5.0.8`之后版本：
```zsh
zsh --version
```
- 设置默认`shell`，需要重启生效。
```zsh
chsh -s $(which zsh)
```

## 2. 安装`Oh My Zsh`
以下为三种安装`Oh My Zsh`方法。

|Method|Command|
|------|-------|
|`curl`|`sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`|
|`wget`|`sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`|
|`fetch`|`sh -c "$(fetch -o - https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`|

## 3. `Oh My Zsh`主目录
`Oh My Zsh`安装主目录在`$HOME/.oh-my-zsh`，在`$HOME/.zshrc`这个文件下可以控制`Oh My Zsh`主要变量。

## 4. 插件
`$HOME/.oh-my-zsh/plugins`这个目录是插件安装的位置，在该目录下每个插件目录为对应的插件名，在`$HOME/.zshrc`文件下设置应用插件。
> [!TIP]
> 插件变量名的值是一个数组类型，可以设置多个插件。

```zsh
plugins=(git python docker tmux)
```

## 5. 主题
`$HOME/.oh-my-zsh/themes`为存储`Oh My Zsh`主题的目录，`GitHub`有[预览主题](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)。
```zsh
ZSH_THEME="agnoster"
```

## 6. 必装外置插件
我最喜欢我的两个外置插件：
1. `zsh-autosuggestions`: 根据输入历史，用浅灰色在光标后预测并补全命令。按 → 键（或自定义快捷键）直接采纳。
```zsh
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```
2. `zsh-syntax-highlighting`: 实时高亮你正在输入的命令。命令存在显示绿色，拼写错误/命令不存在显示红色，字符串和参数会有不同颜色区分。
```zsh
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```
> [!TIP]
> 该插件名必须在`plugins=(...)`最后一位。
