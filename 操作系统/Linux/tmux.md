# tmux命令参考
tmux是一个终端复用器，可以在一个终端内切出多个`plane`并且可以在这些多个`plane`之间移动。也可以作为后台运行某个应用程序。
## 三大层级概念
- Session(会话): 一个独立的tmux工作区，包含多个窗口，退出当前连接后`session`仍然存在。
- Window(窗口)：类似于浏览器的标签页（Tab），一个会话中可以有多个窗口。
- pane(窗格): 一个窗口可以水平、垂直分割成多个窗格。
## 终端命令
在正常的`shell`下执行CMD。
|操作                    |命令                           |
|------------------------|-------------------------------|
|新建会话                |`tmux`                         |
|新建一个带有会话名的会话|`tmux new -s <session-name>`   |
|列出所有会话            |`tmux ls`                      |
|连接会话                |`tmux attach -t <session-name>`|
|关闭/杀死指定会话       |`tmux kill-session -t <session-name>`|
|杀死所有会话            |`tmux kill-server`|
