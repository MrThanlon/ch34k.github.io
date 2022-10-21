---
title: TimeMachine服务器-netatalk
tags: macOS
categories:
  - GNU/Linux
  - 软件
abbrlink: 2811
date: 2019-07-11 23:18:52
---

其实真的不麻烦，也不用samba什么的，直接一个netatalk就完了。

```shell
#ubuntu下
sudo apt-get install -y netatalk
```

然后编辑`/etc/netatalk/AppleVolumes.default`，把带有`"Home Directory"`的那行去掉（应该是180行），加上

```shell
<你用来备份的目录>     "TimeMachine"   volsizelimit:600000     options:tm
```

然后重启一下netatalk

```shell
sudo service netatalk restart
```

或者用`systectl`

```shell
sudo systemctl restart netatalk
```

做好之后在Finder中按`command+k`，输入`afp://<你的服务器ip>`，输入服务器用户名密码，如果顺利的话就能在TimeMachine设置里面看到这个磁盘了。