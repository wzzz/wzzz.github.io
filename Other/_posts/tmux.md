
tmux配置文件：
```
#修改此文件后需要在tmux的mode下加载文件
#source-file ~/.tmux.conf
# 线上环境用红色标识
#set -g status-style "bg=red,fg=cyan"
set -g status-style "fg=red,bold,noreverse"
set -g status-justify centre
set -g status-right ""
set -g status-left ""
set-window-option -g mode-keys vi
set-option -s set-clipboard off
```
