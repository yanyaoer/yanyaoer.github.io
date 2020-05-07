---
title: send notification when task finish
date: 2015-10-11 00:38:23
tags: [tips, notify]
---

终端里运行长时间任务(比如 `make systemimage`)的时候经常会切换到其他环境做别的事情
容易忘记查看之前的任务是否完成, 查到一些方法用在任务结束时发出通知


```bash
#C-z 切到后台运行
fg; tput bel

# Mac OS X
#系统弹窗
osascript -e 'tell app "System Events" to display alert "Build Completed" message "The checkout and build have completed."'

say "Job finished" #语音播报

#notification center
osascript -e 'display notification "Job finished" with title "Alert"'
sudo gem install terminal-notifier
terminal-notifier -message "Job finished!" -title "Alert"

# Ubuntu
notify-send "Job finished!"

# KDE
kdialog --passivepopup 'Job finished'
```


还有 [iterm2 trigger][1] 也能用来触发通知, 高亮文字

## links
[1]: https://iterm2.com/documentation-triggers.html
<http://apple.stackexchange.com/questions/9412/how-to-get-a-notification-when-my-commands-are-done>
<http://superuser.com/questions/345447/how-can-i-trigger-a-notification-when-a-job-process-ends>
