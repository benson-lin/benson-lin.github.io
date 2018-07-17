---
title: "tmux命令 "
layout: post
date: 2018-07-17
tag:
- Linux
categories: LINUX
blog: true
---

# tmux 

prefix = ctrl+b/a

### 基础命令

新建会话：tmux new -s name

断开会话：ctrl+b d，会话会继续在后台运行（tmux detach ）

进入之前断开的会话：tmux a -t name，tmux a回到上一次会话

关闭会话：tmux kill-session -t demo，在当前会话退出可以用exit命令

关闭服务器/关闭所有会话：tmux kill-server

查看所有会话：tmux ls

重命名：prefix + $

在会话中切换会话：ctrl+b s，选择需要的会话回车

新增面板：prefix + ""。prefix + %

面板切换：prefix+方向键

面板最大化：prefix +z

新建窗口：prefix + c。prefix+p上一个窗口，+n下一个窗口。prefix+w选择窗口，+&关闭窗口，+,重命名，+数字切换到对应窗口。+f搜索，支持模糊搜索	

复制文本：在开启鼠标的前提下，长按住shift选择。粘贴时也是按住shift再粘贴



### 灵活配置



新增或修改~/.tmux.conf

如何生效：ctrl+b : source-file ~/.tmux.conf 回车

修改指令前缀：用ctrl+a 替换 ctrl+b

```
set -g prefix C-a #
unbind C-b # C-b即Ctrl+b键，unbind意味着解除绑定
bind C-a send-prefix # 绑定Ctrl+a为新的指令前缀

# 从tmux v1.6版起，支持设置第二个指令前缀
set-option -g prefix2 ` # 设置一个不常用的`键作为指令前缀，按键更快些
```

会话中加载修改后的文件，后面就可以用 prefix+r了

```
# 绑定快捷键为r
bind r source-file ~/.tmux.conf \; display-message "Config reloaded.."
```

新增面板快捷键修改

```
unbind '"'
bind - splitw -v -c '#{pane_current_path}' # 垂直方向新增面板，默认进入当前目录
unbind %
bind | splitw -h -c '#{pane_current_path}' # 水平方向新增面板，默认进入当前目录
```

开启鼠标选择特定窗口和拖动窗口大小

```
setw -g mode-mouse on # 支持鼠标选取文本等
setw -g mouse-resize-pane on # 支持鼠标拖动调整面板的大小(通过拖动面板间的分割线)
setw -g mouse-select-pane on # 支持鼠标选中并切换面板
setw -g mouse-select-window on # 支持鼠标选中并切换窗口(通过点击状态栏窗口名称)

# v2.1及其之后
set-option -g mouse on # 等同于以上4个指令的效果
```

不喜欢使用鼠标的，用方向键也不好用，可以修改快捷键选择面板

```
# 绑定hjkl键为面板切换的上下左右键
bind -r k select-pane -U # 绑定k为↑
bind -r j select-pane -D # 绑定j为↓
bind -r h select-pane -L # 绑定h为←
bind -r l select-pane -R # 绑定l为→

bind -r e lastp # 选择最后一个面板
bind -r ^e last # 选择最后一个窗口

bind -r ^u swapp -U # 与前一个面板交换位置
bind -r ^d swapp -D # 与后一个面板交换位置
```

面板大小

```
# 绑定Ctrl+hjkl键为面板上下左右调整边缘的快捷指令
bind -r ^k resizep -U 10 # 绑定Ctrl+k为往↑调整面板边缘10个单元格
bind -r ^j resizep -D 10 # 绑定Ctrl+j为往↓调整面板边缘10个单元格
bind -r ^h resizep -L 10 # 绑定Ctrl+h为往←调整面板边缘10个单元格
bind -r ^l resizep -R 10 # 绑定Ctrl+l为往→调整面板边缘10个单元格
```

