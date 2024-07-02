# 介绍

前人种树，后人乘凉。本贴所有内容来自互联网以及论坛里各位而我得内容也是站在巨人的肩膀上得出，有这些大佬辛勤付出所得，才有的我们使用的方法和固件，由衷感谢这些前辈们。
本贴主要是给自己做记录以及内容汇总，如果有人遇到同样的问题可以快速的定位和解决，非小白向教程。暂未更完，可先行收藏。

为区分暗云UBOOT 和 官方UBOOT 区别下文均以AUBOOT 及GUBOOT 进行说明解释。不管是暗云的UBOOT 还是官方的UBOOT 均可在 TTL UBOOT 中刷写 CDT 文件。

## 快速了解流程

教程中提供了两种方法写入 uboot 和cdt 文件，两种方法均可按需选择。

AX-16/ZN-M2 列出两种刷机方法，优先推荐通过系统升级刷入openwrt 过渡包进行刷机。

第6点为可选项，如果没有升级硬改过内存跳过即可

刷机方式这里介绍两种，如下所示。拆机备份过程非必须，建议在折腾之前做好固件以及原厂数据备份。请记住，在进行任何刷机操作之前，请确保您对设备和相关操作有足够的了解，并谨慎操作，以避免损坏设备或数据丢失。

- 系统升级方式刷机流程：
  - 1. 拆机
  - 2. 备份原始固件
  - 3. 使用原厂固件升级至过渡固件
  - 4. 在过渡固件中刷写AUBOOT引导程序
  - 5. 重启至AUBOOT并刷写对应的OpenWrt固件
  - 6. 硬改在改好内存后，通过openwrt 刷入 CDT 配置文件

- TTL方式刷机流程：
  - 1. 拆机
  - 2. 备份原始固件
  - 3. 进入GUBOOT
  - 4. 在GUBOOT中刷写AUBOOT引导程序
  - 5. 重启至AUBOOT并刷写对应的OpenWrt固件
  - 6. 硬改在改好内存后，通过AUBOOT 刷入 CDT 配置文件

## 备份原厂系统及数据

使用1.8V ttl 线连接到路由器，打开串口通信软件 putty 或者 sscom 均可，推荐putty

设置并检查环境变量

```bash
printenv
setenv serverip 192.168.10.100

# 备份所有分区，输入以下指令，一行一条

nand read 0x44000000 0x0 0x180000
tftpput 0x44000000 0x180000 SBL1.bin

nand read 0x44000000 0x180000 0x100000
tftpput 0x44000000 0x100000 MIBIB.bin

nand read 0x44000000 0x280000 0x80000
tftpput 0x44000000 0x80000 BOOTCONFIG.bin

nand read 0x44000000 0x300000 0x80000
tftpput 0x44000000 0x80000 BOOTCONFIG1.bin

nand read 0x44000000 0x380000 0x380000
tftpput 0x44000000 0x380000 QSEE.bin

nand read 0x44000000 0x700000 0x380000
tftpput 0x44000000 0x380000 QSEE_1.bin

nand read 0x44000000 0xa80000 0x80000
tftpput 0x44000000 0x80000 DEVCFG.bin

nand read 0x44000000 0xb00000 0x80000
tftpput 0x44000000 0x80000 DEVCFG_1.bin

nand read 0x44000000 0xb80000 0x80000
tftpput 0x44000000 0x80000 RPM.bin

nand read 0x44000000 0xc00000 0x80000
tftpput 0x44000000 0x80000 RPM_1.bin

nand read 0x44000000 0xc80000 0x80000
tftpput 0x44000000 0x80000 CDT.bin

nand read 0x44000000 0xd00000 0x80000
tftpput 0x44000000 0x80000 CDT_1.bin

nand read 0x44000000 0xd80000 0x80000
tftpput 0x44000000 0x80000 APPSBLENV.bin

nand read 0x44000000 0xe00000 0x180000
tftpput 0x44000000 0x180000 APPSBL.bin

nand read 0x44000000 0xf80000 0x180000
tftpput 0x44000000 0x180000 APPSBL_1.bin

nand read 0x44000000 0x1100000 0x80000
tftpput 0x44000000 0x80000 ART.bin

nand read 0x44000000 0x1180000 0x6080000
tftpput 0x44000000 0x6080000 rootfs.bin

nand read 0x44000000 0x7200000 0x80000
tftpput 0x44000000 0x80000 ETHPHYFW.bin

nand read 0x44000000 0x7280000 0xa40000
tftpput 0x44000000 0xa40000 CTCCFW.bin
```

## 刷入OpenWrt

注意：某些机器会出现升级不起作用的情况，具体什么问题暂时不清楚。解决方法是通过TTL 在UBOOT 中刷入。可直接跳过写入过渡固件这一步到刷入UBOOT 这一步，通过TTL-UBOOT 的方方法 刷入暗云大佬的UBOOT，如果硬改好了也可以直接刷CDT 文件
接好TTL 线开机时按回车键进入 UBOOT 命令行界面

### 刷写过渡固件

将过度固件在原厂系统升级界面不保存配置升级即可。

### 刷写UBOOT

暗云大佬发布的uboot 有3个文件，共两个uboot版本，具体作用看下面表格

|文件名|对应分区|备注|
|---|----|---|
|ax18-mibib.bin|0:MIBIB|分区扩容文件|
|uboot-cmiot-ax18.bin|0:APPSBL，0:APPSBL|默认内存版本即256M|
|uboot-cmiot-ax18-mod.bin|0:APPSBL，0:APPSBL_1|扩容内存版本即刷CDT后的512M,1G,2G|

下载地址：<https://mbd.pub/o/bread/YpaZlp5u>

比较推荐直接使用合并分区的uboot 这样可以有更多空间安装插件，这里提供两个方法刷入，可以通过Uboot 写入或者 openwrt 过渡固件写入。

#### 通过过渡固件刷入

以下UBOOT选择一种刷入即可，请结合上面表格自行选择

```bash
# 只刷uboot不合并分区
mtd write /tmp/uboot-cmiot-ax18.bin /dev/mtd13

# 刷uboot合并分区：rootfs 分区达 96m
mtd write /tmp/ax18-mibib.bin /dev/mtd1
mtd write /tmp/uboot-cmiot-ax18-mod.bin /dev/mtd13
```

#### 通过TLL-UBOOT 刷入

以下UBOOT选择一种刷入即可，请结合上面表格自行选择

```bash
# 只刷UBOOT
tftpboot uboot-cmiot-ax18.bin && flash 0:APPSBL
tftpboot uboot-cmiot-ax18.bin && flash 0:APPSBL_1

# 合并和UBOOT
tftpboot ax18-mibib.bin && flash 0:MIBIB
tftpboot uboot-cmiot-ax18-mod.bin && flash 0:APPSBL
tftpboot uboot-cmiot-ax18-mod.bin && flash 0:APPSBL_1
```

### 固件推荐

      以下收集到的几个固件可以尝试，这里推荐第一个固件，sdf8057 大佬的固件无WIFI 温度表现比较好，适合丢弱电箱做软路由使用。

      相关参考：
      https://www.right.com.cn/forum/thread-8286273-1-1.html
      https://github.com/sdf8057/cloudbuild/releases  此固件比编译了无线的固件温度低10度，适合放弱电箱。
      https://anclark.github.io/2023/05/28/OpenWRT/OpenWRT_ZN-M2
      https://www.right.com.cn/forum/thread-8262012-1-1.html

## 硬改

拆机图：<https://www.acwifi.net/13867.html>
资料来源：<https://www.mydigit.cn/thread-335771-1-1.html>

### USB 3.0 原件补齐

      以下提供了一个BOM 清单，可按需在淘宝购买所需原件。

|序号 | 类型 | 型号 | 数量 |
|----|----|----|----|
|1 |DC-DC| TMI3258 | 1 |
|2 |电感 | WHC0530-4.7uH | 1 |
|3 |电阻 | R0603_0R | 1 |
|4 |电阻 | R0805_0R | 2 |
|5 |电阻 | R0201_100K | 1 |
|6 |电阻 | R0201_0R | 2 |
|7 |电阻 | R0201_7.68K | 1 |
|8 |电阻 | R0201_40.2K | 1 |
|9 |电容 | C0603_22uF_16V | 3 |
|10 |电容 | C0201_0.1uF_16V | 2 |
|11 |电容 | C0201_33pF | 1 |
|12 |电容 | C0402_0.1uF | 4 |

命名规范以及注意事项：

DC-DC型号：TMI3258

电感型号：WHC0530-4.7uH

电阻型号：R + 封装 + 阻值（单位欧姆）

电容型号：C + 封装 + 电容值（单位微法） + 电压值（单位伏特）

USB3.0信号：两个组对调一下即可，切记组内信号不得对调，电容是0402 0.1uF*4

### 硬改内存

CDT 内存配置文件

CDT 必须拆机硬改内存更换内存颗粒后才可以刷，否则会砖头！

OpenWrt下刷入:

使用Winscp 将 CDT 文件上传至ZN-M2 的  /tmp  文件夹下，通过Putty 或者SSH工具，连接后输入以下指令

```bash

# 以512M为例
mtd write /tmp/cdt-AX18_AX18_512M.bin /dev/mtd10
mtd write /tmp/cdt-AX18_AX18_512M.bin /dev/mtd11

TTL-UBOOT刷入:
      打开TFTP 工具，将 CDT 文件放在 TFTP 根目录，通过Putty 或者其他串口通信工具，连接TTL 输入以下指令

# 以512M为例
tftpboot cdt-AX18_AX18_512M.bin && flash 0:CDT
tftpboot cdt-AX18_AX18_512M.bin && flash 0:CDT_1

```

## 刷回官方固件

可能有的小伙伴们，刷机是想拿来做 AP  或者对无线有刚需的，体验过OpenWrt固件后发现并不能满足自己的需求，要回官方。这里也提供一个回官方固件的方法（未验证理论可行谨慎操作！）

使用USB-TTL 工具，连接至路由器。打开TFTP 工具将官方固件备份文件放置在TFTP目录下，使用以下命令刷回官方

```bash
tftpboot MIBIB.bin && flash 0:MIBIB
tftpboot APPSBL.bin && flash 0:APPSBL
tftpboot APPSBL.bin && flash 0:APPSBL_1
tftpboot rootfs.bin && flash rootfs
```

## 常见问题

- 在系统升级界面升级不起作用

  - 具体表现形式为：升级中没有任何报错，路由器重启后还是进入原厂固件。可以使用TTL 线 连接至路由器，将TFTP 软件打开，使用TTL 方式刷入系统。
  - 具体流程为进入 UBOOT 中使用TTL 线 刷入UBOOT 然后直接重启至UBOOT 刷 OpenWrt 第三方固件

- 硬改内存后重启/死机

  - 具体表现形式为: 内存占用到某个大小时会卡死，重启，或者出现其他BUG。总之是一些不稳定特征，重启后又表现正常，运行一段时间又复发。
  - 以下解决方案仅针对于焊接工艺达标，没出现虚焊，颗粒正常情况下。使用我上面提供的这个CDT 文件重刷CDT 分区。
  - CDT 文件来自网络，如有侵权联系删除。

- 整机备份及编程器固件

这里也向各位坛友们提供一套固件，需要自取。

  - 编程器固件：<https://rcco.lanzouj.com/izBP223bu2od>
  - 整机单分区备份：https:wwjo.lanzouk.com/ibeiI12h0hcf
