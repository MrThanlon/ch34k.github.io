---
title: 记一次LVM误操作引发的事故
abbrlink: 27038
date: 2021-01-08 21:42:27
tags:
  - 小记
  - GNU/Linux
  - LVM
categorys:
  - GNU/Linux
  - LVM
---

总之是闯了大祸，好在最后数据都没事。主要问题是我对LVM操作不熟练，以及对文件系统和磁盘没有比较清晰的认识，具体来说就是我以为~~LVM在进行lvresize时会自动处理文件系统~~，这当然是错误的，**文件系统应该使用resize2fs命令来调整，LVM管理的是块设备**，不会处理文件系统上的问题。

<!--more-->

## 过程

事情的起因是论坛502了，一般是请求量过大，过一会就好了，但是这次一直持续了8分钟，然后我就上服务器看了下，nginx和php都跑着，挂载根目录的磁盘满了，老规矩删日志，删完还剩500M左右，我就想着能不能处理一下，不然不一会儿又满了。

服务器使用了LVM，好像有硬件RAID（但是操作系统上好像没有对应的软件查看），卷组的大小是1.1T左右，分出了三个逻辑卷

- home卷，总容量1.01T左右，可用802G，挂载到/home目录
- root卷，总容量50G，可用空间告急，挂载到根目录
- swap分区，总容量31G

具体的分配情况如下

```shell
[root@guangdian lib]# vgdisplay
  --- Volume group ---
  VG Name               centos
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <1.09 TiB
  PE Size               4.00 MiB
  Total PE              285647
  Alloc PE / Size       285646 / <1.09 TiB
  Free  PE / Size       1 / 4.00 MiB
  VG UUID               4NKz0b-Alp0-gQJF-LmsB-R0JA-0WhR-RCzalL
[root@guangdian lib]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/centos/swap
  LV Name                swap
  VG Name                centos
  LV UUID                fUURt3-Ee42-aOiy-ZnWF-IOfc-aJ7p-JYVfIf
  LV Write Access        read/write
  LV Creation host, time guangdian, 2017-12-27 16:32:12 +0800
  LV Status              available
  # open                 2
  LV Size                <31.44 GiB
  Current LE             8048
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/centos/home
  LV Name                home
  VG Name                centos
  LV UUID                xeRdqP-oCOa-CzK7-V8Si-qjmM-eUYX-cGlj59
  LV Write Access        read/write
  LV Creation host, time guangdian, 2017-12-27 16:32:13 +0800
  LV Status              available
  # open                 1
  LV Size                1.01 TiB
  Current LE             264798
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
   
  --- Logical volume ---
  LV Path                /dev/centos/root
  LV Name                root
  VG Name                centos
  LV UUID                BPsadR-H2iE-OgiL-t2DB-jL7S-m0Fq-N78Hby
  LV Write Access        read/write
  LV Creation host, time guangdian, 2017-12-27 16:32:30 +0800
  LV Status              available
  # open                 1
  LV Size                50.00 GiB
  Current LE             12800
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```

然后我就抽风了，先用resize2fs压缩，失败，因为文件系统是XFS不支持压缩（另外GFS2也不支持压缩，EXT4支持压缩），但是我没理，直接把home卷给整到300G

```shell
lvresize -L 300G /dev/mapper/centos-home 
```

此时LVM发出警告

```shell
  WARNING: Reducing active and open logical volume to 300.00 GiB.
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce centos/home? [y/n]:
```

我看都没看直接一个y，分配变成了

```shell
[root@guangdian lib]# vgdisplay
  --- Volume group ---
  VG Name               centos
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <1.09 TiB
  PE Size               4.00 MiB
  Total PE              285647
  Alloc PE / Size       97648 / <381.44 GiB
  Free  PE / Size       187999 / 734.37 GiB
  VG UUID               4NKz0b-Alp0-gQJF-LmsB-R0JA-0WhR-RCzalL
[root@guangdian lib]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/centos/swap
  LV Name                swap
  VG Name                centos
  LV UUID                fUURt3-Ee42-aOiy-ZnWF-IOfc-aJ7p-JYVfIf
  LV Write Access        read/write
  LV Creation host, time guangdian, 2017-12-27 16:32:12 +0800
  LV Status              available
  # open                 2
  LV Size                <31.44 GiB
  Current LE             8048
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/centos/home
  LV Name                home
  VG Name                centos
  LV UUID                xeRdqP-oCOa-CzK7-V8Si-qjmM-eUYX-cGlj59
  LV Write Access        read/write
  LV Creation host, time guangdian, 2017-12-27 16:32:13 +0800
  LV Status              available
  # open                 1
  LV Size                300.00 GiB
  Current LE             76800
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
   
  --- Logical volume ---
  LV Path                /dev/centos/root
  LV Name                root
  VG Name                centos
  LV UUID                BPsadR-H2iE-OgiL-t2DB-jL7S-m0Fq-N78Hby
  LV Write Access        read/write
  LV Creation host, time guangdian, 2017-12-27 16:32:30 +0800
  LV Status              available
  # open                 1
  LV Size                50.00 GiB
  Current LE             12800
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```

然后把root卷修改到200G

```shell
lvresize -L 200G /dev/mapper/centos-root
```

分配变成了

```shell
[root@guangdian lib]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/centos/swap
  LV Name                swap
  VG Name                centos
  LV UUID                fUURt3-Ee42-aOiy-ZnWF-IOfc-aJ7p-JYVfIf
  LV Write Access        read/write
  LV Creation host, time guangdian, 2017-12-27 16:32:12 +0800
  LV Status              available
  # open                 2
  LV Size                <31.44 GiB
  Current LE             8048
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/centos/home
  LV Name                home
  VG Name                centos
  LV UUID                xeRdqP-oCOa-CzK7-V8Si-qjmM-eUYX-cGlj59
  LV Write Access        read/write
  LV Creation host, time guangdian, 2017-12-27 16:32:13 +0800
  LV Status              available
  # open                 1
  LV Size                300.00 GiB
  Current LE             76800
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
   
  --- Logical volume ---
  LV Path                /dev/centos/root
  LV Name                root
  VG Name                centos
  LV UUID                BPsadR-H2iE-OgiL-t2DB-jL7S-m0Fq-N78Hby
  LV Write Access        read/write
  LV Creation host, time guangdian, 2017-12-27 16:32:30 +0800
  LV Status              available
  # open                 1
  LV Size                200.00 GiB
  Current LE             51200
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```

直到这时我才发现home卷出问题了，/home无法访问，提示输入输出错误，然后尝试回滚，将分配情况尽可能改回原来的状态，先把多分配给root卷的部分改回去，

```shell
lvresize -L 50GB /dev/mapper/centos-root
```

然后把home卷也改回去，但是我这里失误了，命令是

```shell
lvresize -L 802GB /dev/mapper/centos-home
```

不应该是802G，而是1.01T左右，然后我又执行了命令

```shell
lvresize -L 1.01TiB /dev/mapper/centos-home
```

然后中途网络波动，ssh断了，服务器是仅公钥登陆，在无法访问/home的情况下，我远程登不上，只能进机房操作了。

## 恢复

当时我在准备DSP和高频的考试，是豆豆帮我去沙河机房把磁盘整个拷贝了出来，十分感谢豆豆的帮助。

然后我们是在一台Debian Live CD的系统上尝试恢复的。将磁盘镜像挂载到回环设备上之后，LVM识别到了卷组。

重新整理一下上面操作过程对LVM的影响，好在VG分配是线性的，用LE（logical extent）作为单位精确描述一下，

0. 初始
   - swap: 8048, [0, 8047]
   - home: 264798, [8048, 272845]
   - root: 12800, [272846, 285645]
1. 第一次lvresize命令，home压缩到300G
   - swap: 同上
   - home: 76800, [8048, 84847]
   - root: 同上
2. 第二次lvresize命令，root扩展到200G
   - swap: 同上
   - home: 同上
   - root: 大小从12800变成51200，增加了38400，即加上了一个段[84847, 123246]
3. 第三次lvresize命令，root还原回50G
   - swap: 同上
   - home: 同上
   - root: 还原到12800，返回到第一次lvresize时候的状况
4. 第四次lvresize命令，home变为802G
   - swap: 同上
   - home: 因为没有查看分配情况，实际是啥不得而知
   - root: 同上
5. 第五次lvresize命令，home变为1.01T
   - swap: 同上
   - home: 在之后从磁盘镜像中看到的情况来看，LE数量为264766，[8048, 272813]，相比最初的分配情况少了32个LE
   - root: 不变

具体过程就是这样了，考虑到并没有执行xfs_growfs，应该没有额外的数据被写入到不适合的位置，所以最后直接将home改回原来的位置应该就行。

```shell
lvresize -L +32 /dev/mapper/centos-home
```

服务器重启之后/home能正常访问，网站也恢复了。

这部分是在涛哥的指导下进行的，包括整个过程的分析，以及最后的+32，感谢涛哥对我的忍耐。

## 总结

作为运维，应该要明确知道敲下的命令会发生什么，会有什么严重后果，如果确实不清楚，就去看手册、搜索引擎。如果无法避免严重的后果，就应该备份好重要数据。