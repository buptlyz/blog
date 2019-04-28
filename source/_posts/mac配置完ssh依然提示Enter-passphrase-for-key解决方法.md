---
title: mac配置完ssh依然提示Enter passphrase for key解决方法
date: 2019-04-28 22:09:11
tags: git
---

已经配置git ssh key，但在git pull/push的时候仍然需要输入密码，并且提示：

    Enter passphrase for key 'xxxx'

解决办法：

输入命令 

    ssh-add -K xxx
