---
title: 在树莓派上使用SATA设备(ASM1062/ASM1061)
abbrlink: 48167
date: 2021-01-30 20:33:50
tags:
  - GNU/Linux
  - 树莓派
categories:
  - 嵌入式开发
  - 树莓派
---

众所周知，树莓派4使用的BCM2711拥有一个千兆以太网口，以及一条PCIe2.0 1x，也就是说如果我们把PCIe转接到SATA，就可以用树莓派当NAS了，岂不美哉？

<!--more-->

当然，对于树莓派4B来说，它的PCIe已经分配给了USB3.0，所以我们这次要用的是树莓派CM4，它引出了大多数引脚，包括以太网口和PCIe。如果是自己画板子，要注意PCIe是差分对，需要等长布线，我这里为了方便直接买了官方出的底板，一样的用法。

淘宝上比较便宜的方案是ASM1061/ASM1062，这俩都能引出两个SATA口，而且Linux已经包含了它的驱动程序，直接就能使用。关于这玩意的讨论可以看看[这个issue](https://github.com/geerlingguy/raspberry-pi-pcie-devices/issues/30)。

买到这个卡之后直接插上开机，输入`lspci`就直接能看到ASM1062，如下

```shell
 pi@raspberrypi  ~  sudo lspci   
00:00.0 PCI bridge: Broadcom Limited Device 2711 (rev 20)
01:00.0 SATA controller: ASMedia Technology Inc. ASM1062 Serial ATA Controller (rev 01)
```

但是如果我们直接连上硬盘，可以发现`/dev/`里面并没有硬盘设备，这是因为树莓派官方的操作系统里面的内核并没有把SATA驱动也编译进去，所以我们只好自己重新编译并安装了。

首先我们需要安装编译需要的工具，如果是在树莓派上编译的话，可以运行以下命令安装

```shell
sudo apt install -y git bc bison flex libssl-dev make libncurses5-dev build-essential
```

如果是在电脑上交叉编译，那就自己看着安装吧。

然后需要克隆linux源码到本地，这里我们使用树莓派基金会维护的linux

```shell
git clone --depth=1 https://github.com/raspberrypi/linux
```

然后进入到目录进行编译前的配置，首先添加环境变量

```shell
export KERNEL=kernel7l
```

如果是要编译64bit的内核就改成`kernel8`，32bit就如上`kernel7l`。

另外如果是在电脑上交叉编译，还需要安装交叉编译器，可以使用[树莓派仓库](https://github.com/raspberrypi/tools)里的那个，把bin目录添加到PATH，同时还要添加一个CROSS_COMPILE的环境变量

```shell
export CROSS_COMPILE=arm-linux-gnueabihf-
```

然后deconfig一下

```shell
make bcm2711_defconfig
```

然后我们需要修改.config文件，可以用menuconfig来完成，如下

```shell
make menuconfig
```

在菜单中找到Device Drivers，按回车进入，找到Serial ATA and Parallel ATA drivers (libata)，按一下空格选中，然后按回车进入里面找到AHCI SATA support，也按一下空格选中。如果你也是ASM106x芯片的卡，那么到这里就完成了，如果是Marvell芯片的，就把Marvell SATA support也加上，在那一行按空格即可，最后保存为.config文件并退出。

如果不想用menuconfig也可以手动修改.config文件，主要是这些地方要修改

```
CONFIG_ATA=m
CONFIG_ATA_VERBOSE_ERROR=y
CONFIG_SATA_PMP=y
CONFIG_SATA_AHCI=m
CONFIG_SATA_MOBILE_LPM_POLICY=0
CONFIG_ATA_SFF=y
CONFIG_ATA_BMDMA=y
CONFIG_SATA_MV=m
```

最后可以修改下Localversion，在.config文件中找到CONFIG_LOCALVERSION，加个后缀啥的，比如`-v7l-sata`之类的，然后就可以开始编译啦

```shell
make -j4 zImage modules dtbs
```

编译好之后安装替换现在的内核，重启就完事了

```shell
sudo make modules_install
sudo cp arch/arm/boot/dts/*.dtb /boot/
sudo cp arch/arm/boot/dts/overlays/*.dtb* /boot/overlays/
sudo cp arch/arm/boot/dts/overlays/README /boot/overlays/
sudo cp arch/arm/boot/zImage /boot/$KERNEL.img
```

最后附上我自己用一个SSD测试的结果

```
 pi@raspberrypi  ~  sudo fdisk -l /dev/sda
Disk /dev/sda: 55.9 GiB, 60022480896 bytes, 117231408 sectors
Disk model: GLOWAY FER60GS3-
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
 pi@raspberrypi  ~  sudo smartctl -a /dev/sda
smartctl 6.6 2017-11-05 r4594 [armv7l-linux-5.4.83-v7l+] (local build)
Copyright (C) 2002-17, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF INFORMATION SECTION ===
Device Model:     GLOWAY FER60GS3-S7
Serial Number:    DA3D05C020ACC0010047
LU WWN Device Id: 5 000000 000000000
Firmware Version: F1AC20
User Capacity:    60,022,480,896 bytes [60.0 GB]
Sector Size:      512 bytes logical/physical
Rotation Rate:    Solid State Device
Device is:        Not in smartctl database [for details use: -P showall]
ATA Version is:   ACS-2 (minor revision not indicated)
SATA Version is:  SATA 3.1, 6.0 Gb/s (current: 6.0 Gb/s)
Local Time is:    Sat Jan 30 12:39:45 2021 GMT
SMART support is: Available - device has SMART capability.
SMART support is: Enabled

=== START OF READ SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED

General SMART Values:
Offline data collection status:  (0x00)	Offline data collection activity
					was never started.
					Auto Offline Data Collection: Disabled.
Self-test execution status:      (   0)	The previous self-test routine completed
					without error or no self-test has ever 
					been run.
Total time to complete Offline 
data collection: 		(    0) seconds.
Offline data collection
capabilities: 			 (0x71) SMART execute Offline immediate.
					No Auto Offline data collection support.
					Suspend Offline collection upon new
					command.
					No Offline surface scan supported.
					Self-test supported.
					Conveyance Self-test supported.
					Selective Self-test supported.
SMART capabilities:            (0x0002)	Does not save SMART data before
					entering power-saving mode.
					Supports SMART auto save timer.
Error logging capability:        (0x01)	Error logging supported.
					General Purpose Logging supported.
Short self-test routine 
recommended polling time: 	 (   1) minutes.
Extended self-test routine
recommended polling time: 	 (   1) minutes.
Conveyance self-test routine
recommended polling time: 	 (   1) minutes.
SCT capabilities: 	       (0x003d)	SCT Status supported.
					SCT Error Recovery Control supported.
					SCT Feature Control supported.
					SCT Data Table supported.

SMART Attributes Data Structure revision number: 1
Vendor Specific SMART Attributes with Thresholds:
ID# ATTRIBUTE_NAME          FLAG     VALUE WORST THRESH TYPE      UPDATED  WHEN_FAILED RAW_VALUE
  1 Raw_Read_Error_Rate     0x0000   100   100   000    Old_age   Offline      -       0
  5 Reallocated_Sector_Ct   0x0000   100   100   000    Old_age   Offline      -       1
  9 Power_On_Hours          0x0000   100   100   000    Old_age   Offline      -       149
 12 Power_Cycle_Count       0x0000   100   100   000    Old_age   Offline      -       690
160 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       0
161 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       58
163 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       20
164 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       28663
165 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       101
166 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       0
167 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       55
168 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       3000
169 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       99
175 Program_Fail_Count_Chip 0x0000   100   100   000    Old_age   Offline      -       7
176 Erase_Fail_Count_Chip   0x0000   100   100   000    Old_age   Offline      -       0
177 Wear_Leveling_Count     0x0000   100   100   050    Old_age   Offline      -       0
178 Used_Rsvd_Blk_Cnt_Chip  0x0000   100   100   000    Old_age   Offline      -       1
181 Program_Fail_Cnt_Total  0x0000   100   100   000    Old_age   Offline      -       7
182 Erase_Fail_Count_Total  0x0000   100   100   000    Old_age   Offline      -       0
192 Power-Off_Retract_Count 0x0000   100   100   000    Old_age   Offline      -       55
194 Temperature_Celsius     0x0000   100   100   000    Old_age   Offline      -       24
195 Hardware_ECC_Recovered  0x0000   100   100   000    Old_age   Offline      -       282
196 Reallocated_Event_Count 0x0000   100   100   016    Old_age   Offline      -       0
197 Current_Pending_Sector  0x0000   100   100   000    Old_age   Offline      -       0
198 Offline_Uncorrectable   0x0000   100   100   000    Old_age   Offline      -       0
199 UDMA_CRC_Error_Count    0x0000   100   100   050    Old_age   Offline      -       0
232 Available_Reservd_Space 0x0000   100   100   000    Old_age   Offline      -       98
241 Total_LBAs_Written      0x0000   100   100   000    Old_age   Offline      -       52988
242 Total_LBAs_Read         0x0000   100   100   000    Old_age   Offline      -       70158
245 Unknown_Attribute       0x0000   100   100   000    Old_age   Offline      -       114652

SMART Error Log Version: 1
No Errors Logged

SMART Self-test log structure revision number 1
No self-tests have been logged.  [To run self-tests, use: smartctl -t]

SMART Selective self-test log data structure revision number 1
 SPAN  MIN_LBA  MAX_LBA  CURRENT_TEST_STATUS
    1        0        0  Not_testing
    2        0        0  Not_testing
    3        0        0  Not_testing
    4        0        0  Not_testing
    5        0        0  Not_testing
    6        0    65535  Read_scanning was never started
Selective self-test flags (0x0):
  After scanning selected spans, do NOT read-scan remainder of disk.
If Selective self-test is pending on power-up, resume after 0 minute delay.
```

读取速度在260MB/s左右，够用了。