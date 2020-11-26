---
title: Linux 磁盘分区、格式化、挂载
date: 2018-12-18 10:51:51
categories: Linux
tags: [Linux, 分区, 格式化, 挂载]
toc: true
---
通过本文学习 Linux 磁盘分区、格式化、挂载。
<!-- more -->

## 1 查看磁盘分区

fdisk -l 可以列出所有的分区，包括没有挂上的分区和 usb 设备。一般可以用这个来查找需要挂载的分区的位置，比如挂上 u 盘。

```bash
fdisk -l
Disk /dev/xvda: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x000b1054

    Device Boot      Start         End      Blocks   Id  System
/dev/xvda1   *           1          26      204800   83  Linux
Partition 1 does not end on cylinder boundary.
/dev/xvda2              26         548     4194304   82  Linux swap / Solaris
Partition 2 does not end on cylinder boundary.
/dev/xvda3             548        2611    16571392   83  Linux

Disk /dev/xvde: 107.4 GB, 107374182400 bytes
255 heads, 63 sectors/track, 13054 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xc0c19043

    Device Boot      Start         End      Blocks   Id  System
/dev/xvde1               1       13054   104856223+  83  Linux

Disk /dev/xvdf: 53.7 GB, 53687091200 bytes
255 heads, 63 sectors/track, 6527 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000
```

通过上面的信息，我们知道此机器中挂载3个硬盘（或移动硬盘）：xvda、xvde、xvdf。如果我们想查看单个硬盘情况，可以通过 fdisk -l /dev/xvda1 来操作。

- 第1个 **xvda** 有三个主分区（包括扩展分区），分别是主分区 xvda1、xvda2、xvda3。如果有逻辑分区，则会从 xvda5 开始，因为主分区（包括扩展分区）的总个数不能超过4个，也不能把扩展分区包围在主分区之间。
- 第2个 **xvde** 有1个主分区 xvde1。如果有逻辑分区，则会从 xvde5 开始。  
- 第3个 **xvdf** 还未分区，下面会进行分区。

硬盘总容量=主分区（包括扩展分区）总容量。扩展分区容量=逻辑分区总容量。通过上面的例子，我们可以得知 xvda=xvda1+xvda2+xvda3。如果 xvda3 为扩展分区，则 xvda3=xvda3+xvda6+xvda7+......。

说明：

- 硬盘分区的表示：在 Linux 是通过 hd%& 或 sd%& 表示的，上面所列是 xvd%& ，其中 % 表示的是 a、b、c 等，& 表示的数字1、2、3等。hd 大多是 IDE 硬盘；sd 大多是 SCSI 或移动存储；  
- 引导（Boot）：表示引导分区，在上面的例子中 hda1 是引导分区；  
- Start （开始）：表示的一个分区从 X cylinder（磁柱）开始；  
- End （结束）：表示一个分区到 Y cylinder（磁柱）结束；  
- id 和 System 表示的是一个意思，id 看起来不太直观，我们要在 fdisk 一个分区时，通过指定 id 来确认分区类型；比如 7 表示的就 NTFS 分区；这个在 fdisk 中要通过t功能来指定。下面的部分会提到；  
- Blocks（容量）：这是我翻译的，其实不准确，表示的意思的确是容量的意思，其单位是 K；一个分区容量的值是由下面的公式而来的；Blocks = （相应分区 End 数值 - 相应分区 Start 数值）x 单位 cylinder（磁柱）的容量。

我们估算一个硬盘是否完全被划分，我们只要看 fdisk -l 输出的内容中的 cylinders（柱体） 上一个分区的 End 和下一个分区的 Start 是不是一个连续的数字，另外要看一下每个硬盘设备的 fdisk -l 的开头部份，看一下他的 cylinders（柱体）的值。上一个分区的 End 的值 +1 就是下一个分区的 Start 的值。如果看到 End 的值是跟在 fdisk -l 头部信息中 cylinders 的值一样的话，证明这个硬盘已经完全划分。

## 2 磁盘分区

```bash
fdisk /dev/xvdf
Command (m for help): m
　　Command action
　　a toggle a bootable flag
　　b edit bsd disklabel
　　c toggle the dos compatibility flag
　　d delete a partition 注：d 删除一个分区；
　　l list known partition types 注：l 列出分区类型，以供我们设置相应分区的类型；
　　m print this menu 注：m 列出帮助信息；
　　n add a new partition 注：n 添加一个分区；
　　o create a new empty DOS partition table
　　p print the partition table 注：p 列出分区表；
　　q quit without saving changes 注：q 不保存退出；
　　s create a new empty Sun disklabel
　　t change a partition's system id 注：t 改变分区类型；
　　u change display/entry units
　　v verify the partition table
　　w write table to disk and exit 注：w 把分区表写入硬盘保存并退出；
　　x extra functionality (experts only) 注：扩展应用，专家功能；
Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
p 注：添加主分区
Partition number (1-4): 1 注：第一个主分区
First cylinder (1-6527, default 1): 1 注：起始位置，可直接回车默认1
Last cylinder, +cylinders or +size{K,M,G} (1-6527, default 6527): 6527 注：结束位置，可直接回车默认6527，表示磁盘全部空间分到一个分区中。或者输入别的数值，或者也可输+200M来指定分区大小为200M。

Command (m for help): w 注：保存退出，q 不保存退出
```

## 3 分区格式化、挂载

```bash
# 分区格式化
# mkfs - 支持 ext2、ext3（日志）、vfat、msdos、jfs、reiserfs 等
mkfs.ext3 /dev/xvdf1

# 分区挂载
# mount 挂载设备 挂载点
mkdir /usr/local/mydata
mount /dev/xvdf1 /usr/local/mydata
df -hT
Filesystem     Type   Size  Used Avail Use% Mounted on
/dev/xvda3     ext4    16G  4.5G   11G  30% /
tmpfs          tmpfs   32G   80K   32G   1% /dev/shm
/dev/xvda1     ext4   194M   46M  139M  25% /boot
/dev/xvde1     ext3    99G   40G   55G  43% /usr/local/mysqldata
/dev/xvdf1     ext3    53G  976M   50G   2% /usr/local/mydata

# 将挂载点写入注册表，系统启动后自动挂载
vi /etc/fstab
新增一行：/dev/xvdf1(分区) /usr/local/mydata(挂载点) ext3(类型) defaults 0 0

# 分区取消挂载
umount /dev/xvdf1
# 或
umount /usr/local/mydata
```


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)