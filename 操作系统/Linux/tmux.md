# tmux
`tmux`是一个终端复用器，它可以在一个终端程序内分配多个伪终端，并且支持后台支行，即使关闭当前终端，在`tmux`会话内的任务正常运行。

## 三大概念
- **会话(Session)：** `tmux` 会话（session）是 tmux 中最顶层的工作空间概念。它可以理解为一个“持久化的终端环境”，里面可以包含多个窗口（window），每个窗口又可以包含多个面板（pane）。
- **窗口(Window):** 一个独立的终端窗口，相当于浏览器的“标签页”
- **窗格(Panes):** 窗口里面切分出来的小终端，相当于编辑器里的“分屏”
```text
tmux session（会话）
│
├── window 0（窗口）
│      │
│      ├── pane 0（窗格）
│      │       bash
│      │
│      └── pane 1（窗格）
│              top
│
├── window 1（窗口）
│      │
│      └── pane 0
│              vim
│
└── window 2（窗口）
       │
       └── pane 0
              logs
```

## 命令参考
> [!NOTE]
> `tmux`前缀(Prefix)键默认是`Ctrl+c`，可以通过配置文件修改。

`tmux`的命令可以从3个方式发出。1. `Shell`终端发出。2. 在 `tmux` 内部通过命令模式`(:)`发出。3. 直接通过快捷键绑定发出。

### 会话命令
会话是 tmux 层级结构的顶层。一个会话会将多个窗口分组，即使断开连接，它也会继续在后台运行。会话在 SSH 连接断开后仍然存在，并且可以从任何终端重新连接。

|命令|短名|绑定快捷键|描述|
|:-------|:----|:----------|:----|
|`new-session -s name`|`new -s name`|**—**|创建一个命名的会话|
|`list-sessions`|`ls`|**—**|列出所有会话|
|`attach-session -t name`|`attach -t name`|**—**|连接一个己命名的会话|
|`detach-client`|**—**|`Prefix d`|分离当前会话在后台运行|
|`rename-session -t old new`|**—**|`Prefix $`|重命名一个会话|
|`kill-session -t name`|**—**|**—**|杀死一个会话|
|`kill-server`|**—**|**—**|杀死`tmux`服务和所有会话|
|`has-session -t name`|**—**|**—**|检查会话是否存在（返回 0/1）|
|`switch-client -t name`|**—**|`Prefix s`|以交互方式切换到另一个会话|
|`switch-client -p`|**—**|`Prefix (`|切换至上一个会话|
|`switch-client -n`|**—**|`Prefix )`|切换至下一个会话|

### 窗口命令
窗口是会话中的标签页。每个窗口都有自己的一组窗格，并占据整个终端空间。使用窗口可以在同一会话中组织不同的任务。

|命令|绑定快捷键|描述|
|:----|:----------|:----|
|`new-window -n name`|`Prefix c`|创建一个新窗口(可带名称)|
|`rename-window name`|`Prefix ,`|重命名窗口|
|`select-window -t :N`|`Prefix 0-9`|通过窗口索引号选择窗口|
|`next-window`|`Prefix n`|切换下个窗口|
|`previous-window`|`Prefix p`|切换上个窗口|
|`last-window`|`Prefix l`|在最后两个窗口之间切换|
|`list-windows`|`Prefix w`|交互式选择窗口，通过窗口选择器|
|`find-window -N name`|`Prefix f`|通过名称搜索窗口|
|`kill-window -t :N`|`Prefix &`|杀死窗口和所有窗格|
|`move-window -t session:N`|`Prefix .`|将窗口移动到另一个会话/索引|
|`swap-window -s N -t M`|**—**|交换两个窗口的位置|
|`move-window -r`|**—**|重新编号窗口以消除序列中的间隙|

### 窗格命令
窗格是窗口内的分割区域。每个窗格都运行一个独立的界面。窗格可以调整大小、交换位置、缩放比例，也可以在窗口之间移动。