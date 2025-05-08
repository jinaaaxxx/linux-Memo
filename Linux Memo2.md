# 1.磁盘和文件系统管理

## 1.磁盘基础知识

### 1.1 磁盘接口类型

- IDE：133MB/s，并行接口，早期家用电脑
- SCSI：640MB/s，并行接口，早期服务器
- SATA：6Gbps，SATA数据端口与电源端口是分开的，即需要两条线，一条数据线，一条电源线
- SAS：6Gbps，SAS是一整条线，数据端口与电源端口是一体化的，SAS中是包含供电线的，而 SATA中不包含供电线。SATA标准其实是SAS标准的一个子集，二者可兼容，SATA硬盘可以插入 SAS主板上，反之不成
- USB
- M.2

注意：速度不是由单纯的接口类型决定，支持Nvme协议硬盘速度是最快的

### 1.2 硬盘存储术语

##### 1.2.1 CHS

- CHS采用 24 bit位寻址
- 其中前10位表示cylinder，中间8位表示head，后面6位表示sector
- 最大寻址空间 8 GB
- head：磁头 磁头数=盘面数
- track：磁道 磁道=柱面数
- sector：扇区，512bytes
- cylinder：柱面 1柱面=512 * sector数/track*head数=512*63*255=7.84M

##### 1.2.2 LBA（logical block addressing）

- LBA是一个整数，通过转换成 CHS 格式完成磁盘具体寻址
- ATA-1规范中定义了28位寻址模式，以每扇区512位组来计算，ATA-1所定义的28位LBA上限达到 128 GiB。2002年ATA-6规范采用48位LBA，同样以每扇区512位组计算容量上限可达128 Petabytes

由于CHS寻址方式的寻址空间在大概8GB以内，所以在磁盘容量小于大概8GB时，可以使用CHS寻址方 式或是LBA寻址方式；在磁盘容量大于大概8GB时，则只能使用LBA寻址方式

## 2.管理存储

### 2.1 磁盘使用三部曲之磁盘分区

##### 2.1.1 分区的意义

- 优化I/O性能
- 实现磁盘空间配额限制
- 提高修复速度
- 隔离系统和程序
- 安装多个OS
- 采用不同文件系统

##### 2.1.2 MBR分区

MBR：Master Boot Record，1982年，使用32位表示扇区数，**分区不超过2T**

MBR分区前512bytes

 446bytes: boot loader

 64bytes：分区表，其中每16bytes标识一个分区

 2bytes: 55AA（表示分区正在被使用）

MBR分区中一块硬盘最多有4个主分区，也可以3主分区+1扩展(N个逻辑分区)

MBR分区：主和扩展分区对应的1--4，/dev/sda3，逻辑分区从5开始，/dev/sda5

##### 2.1.3 GPT分区

GPT：GUID（Globals Unique Identifiers） partition table 支持128个分区，使用64位，支持8Z（ 512Byte/block ）64Z （ 4096Byte/block）

使用128位UUID(Universally Unique Identifier) 表示磁盘和分区 GPT分区表自动备份在头和尾两份， 并有CRC校验位

UEFI (Unified Extensible Firmware Interface 统一可扩展固件接口)硬件支持GPT，使得操作系统可以启动

GPT分区结构分为4个区域：

- GPT头
- 分区表
- GPT分区
- 备份区域

##### 2.1.4 查看分区

lsblk：列出磁盘设备

```bash
lsblk [options] [device...]
```

查看内核是否已经识别新的分区

```bash
cat /proc/partations
```

若硬盘添加无法显示可执行如下操作

```bash
[root@centos8 data]# ll /sys/block/sd*
lrwxrwxrwx. 1 root root 0 Jan 30 23:41 /sys/block/sda -> ../devices/pci0000:00/0000:00:10.0/host11/target11:0:0/11:0:0:0/block/sda
[root@centos8 data]echo "- - -" > /sys/class/scsi_host/host11/scan
此时可重新查看添加的硬盘
```

##### 2.1.5 fdisk和gdisk创建分区

格式：

```bash
gdisk [device...]             类fdisk 的GPT分区工具
fdisk -l [-u] [device...]     查看分区
fdisk [device...]             管理MBR分区
```

子命令：

```bash
p 分区列表
t 更改分区类型
n 创建新分区
d 删除分区
v 校验分区
u 转换单位
w 保存并退出
q 不保存并退出
```

分区后重读分区表：

```bash
#Centos6 通知内核重新读取硬盘分区表除了CentOS 6 以外的其它版本 5，7，8
partprobe
#Centos6 通知内核重新读取硬盘分区表
新增分区用
partx -a /dev/DEVICE
kpartx -a /dev/DEVICE -f: force

删除分区用
partx -d --nr M-N /dev/DEVICE
```

MBR分区推荐使用fdisk命令，GPT推荐使用gdisk命令



##### 2.1.6 parted 命令分区

注意：parted的操作与fdisk和gdisk不同的是它是实时生效

格式：

```bash
parted [选项]... [设备 [命令 [参数]...]...] 
```

常用选项：

- -l 列出所有硬盘分区信息
- -p 进入交互命令模式

常见命令：

- mklabel label-type 指定分区方式（GPT或者MBR）
- mkpart [part-type name fs-type] start end 创建指定大小的分区，默认单位M
- print 显示所有分区
- rm 删除指定分区
- name 指定分区名字

范例：

```bash
[root@centos8 ~]# parted /dev/sdb mklabel gpt
Information: You may need to update /etc/fstab.
[root@centos8 ~]# parted /dev/sdb print                             
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

[root@centos8 ~]# parted /dev/sdb mkpart primary ext4 1001 2001           
Information: You may need to update /etc/fstab
[root@centos8 ~]# parted /dev/sdb print
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 
Number  Start   End     Size   Type      File system  Flags
 1      1049kB  1000MB  999MB  extended               lba
 2      1001MB  2001MB  999MB  primary

[root@centos8 ~]# parted /dev/sdb rm 1
Information: You may need to update /etc/fstab.
[root@centos8 ~]# parted /dev/sdb print
Model: VMware, VMware Virtual S (scsi)
Disk /dev/sdb: 21.5GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 
Number  Start   End     Size   Type     File system  Flags
 2      1001MB  2001MB  999MB  primary
```

### 2.2 磁盘使用三部曲之创建文件系统

##### 2.2.1 文件系统概念

操作系统中负责管理和存储文件信息的软件结构称为文件管理系统，简称文件系统

查看支持的文件系统：

```bash
/lib/modules/`uname -r`/kernel/fs
cat /proc/filesystems
```

帮助：man 5 fs

##### 2.2.2 文件系统类型

Linux 常用文件系统

- ext2：Extended file system 适用于那些分区容量不是太大，更新也不频繁的情况，例如 /boot 分 区
- ext3：是 ext2 的改进版本，其支持日志功能，能够帮助系统从非正常关机导致的异常中恢复
- ext4：是 ext 文件系统的最新版。提供了很多新的特性，包括纳秒级时间戳、创建和使用巨型文件 (16TB)、最大1EB的文件系统，以及速度的提升
- xfs：SGI，支持最大8EB的文件系统
- swap
- iso9660 光盘
- btrfs（Oracle）

Windows 常用文件系统

- FAT32 ：最多只能支持16TB的文件系统和4GB的文件
- NTFS ：最多只能支持16EB的文件系统和16EB的文件
- exFAT

##### 2.2.3 创建文件系统

mkfs：创建文件系统

mke2fs：ext系列文件系统专用管理工具

格式：

```bash
mkfs [options] [-t type] [fs-options] device [size]
mkfs.type device
mke2fs [options] device
```

常用选项：

- -t {ext2|ext3|ext4|xfs} 指定文件系统类型
- -b {1024|2048|4096} 指定块 block 大小
- -L ‘LABEL’ 设置卷标
- -j 相当于 -t ext3， mkfs.ext3 = mkfs -t ext3 = mke2fs -j = mke2fs -t ext3
- -i # 为数据空间中每多少个字节创建一个inode；不应该小于block大小
- -N # 指定分区中创建多少个inode -I 一个inode记录占用的磁盘空间大小，128---4096
- -m # 默认5%,为管理人员预留空间占总空间的百分比
- -O FEATURE[,...] 启用指定特性
- -O ^FEATURE 关闭指定特性

##### 2.2.4 查看文件系统

blkid：显示设备属性信息

格式：

```bash
blkid [OPTION]... [DEVICE]
```

常用选项：

- -L LABEL 根据指定的LABEL来查找对应的设备
- -U UUID 根据指定的UUID来查找对应的设备

e2label：管理ext系列文件系统的LABEL

格式：

```bash
e2label DEVICE [LABEL]
```

findfs：通过label和UUID查找分区

格式：

```bash
findfs [options] LABEL=<label>
findfs [options] UUID=<uuid>
```

dumpe2fs：显示ext文件系统信息，将磁盘块分组管理

格式：

```bash
dumpe2fs  [options] device
```

常用选项：

- -h：查看超级块信息，不显示分组信息

xfs_info：显示示挂载或已挂载的 xfs 文件系统信息

格式：

```bash
xfs_info mountpoint|devname
```

范例：

```bash
[root@centos8 ~]# xfs_info /dev/sdb1
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=65536 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=262144, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

##### 2.2.5 修复文件系统

文件系统夹故障常发生于死机或者非正常关机之后，挂载为文件系统标记为“no clean”

**注意：一定不要在挂载状态下执行下面命令修复**

fsck: File System Check

格式：

```bash
fsck.FS_TYPE
fsck -t FS_TYPE 
FS_TYPE 一定要与分区上已经文件类型相同
```

常用选项：

- -a 自动修复
- -r 交互式修复错误

e2fsck：ext系列文件专用的检测修复工具

格式：

```bash
-y 自动回答为yes
-f 强制修复
-p 自动进行安全的修复文件系统问题
```

xfs_repair：xfs文件系统专用检测修复工具

格式：

```bash
xfs_repair  [options] device
```

常用选项：

- -f 修复文件
- -n 只检查
- 允许修复只读的挂载设备，在单用户下修复 / 时使用，然后立即reboot

### 2.3 磁盘使用三部曲之挂载文件系统

挂载:将额外文件系统与根文件系统某现存的目录建立起关联关系，进而使得此目录做为其它文件访问入 口的行为

##### 2.3.1 挂载文件系统mount

格式：

```bash
mount [-fnrsvw] [-t fstype] [-o options] device dir
```

device：指明要挂载的设备

- 设备文件：例如:/dev/sda5
- 卷标：-L 'LABEL', 例如 -L 'MYDATA'
- UUID： -U 'UUID'：例如 -U '0c50523c-43f1-45e7-85c0-a126711d406e'
- 伪文件系统名称：proc, sysfs, devtmpfs, configfs

mountpoint：挂载点目录必须事先存在，建议使用空目录

常用选项：

- -t vsftype 指定要挂载的设备上的文件系统类型

- -r readonly，只读挂载

- -w read and write, 读写挂载

- -n 不更新/etc/mtab，mount不可见

- -a 自动挂载所有支持自动挂载的设备(定义在了/etc/fstab文件中，且挂载选项中有 auto功能)

- -B, --bind 绑定目录到另一个目录上

- -o options：(挂载文件系统的选项)，多个选项使用逗号分隔

   async 异步模式,内存更改时,写入缓存区buffer,过一段时间再写到磁盘中，效率高，但不安全

   sync 同步模式,内存更改时，同时写磁盘，安全，但效率低下

   atime/noatime 包含目录和文件

   diratime/nodiratime 目录的访问时间戳

   auto/noauto 是否支持开机自动挂载，是否支持-a选项

   exec/noexec 是否支持将文件系统上运行应用程序

   dev/nodev 是否支持在此文件系统上使用设备文件

   suid/nosuid 是否支持suid和sgid权限

   remount 重新挂载 ro/rw 只读、读写

   user/nouser 是否允许普通用户挂载此设备，/etc/fstab使用

   acl/noacl 启用此文件系统上的acl功能 loop 使用loop设备

   _netdev 当网络可用时才对网络资源进行挂载，如：NFS文件系统 defaults 相当于rw, suid, dev, exec, auto, nouser, async

挂载规则:

- 一个挂载点同一时间只能挂载一个设备
- 一个挂载点同一时间挂载了多个设备，只能看到最后一个设备的数据，其它设备上的数据将被隐藏
- 一个设备可以同时挂载到多个挂载点

##### 2.3.2 卸载文件系统 umount

格式：

```bash
umount 设备名|挂载点
```

##### 2.3.3 查看挂载情况

查看挂载

```bash
#通过文件查看
/etc/fstab, /etc/mtab and /proc/mounts

#通过命令查看
mount
```

查看挂载点情况

```bash
#用法
findmnt [options] device|mountpoint

#范例
[root@centos8 ~]# findmnt /boot
TARGET SOURCE    FSTYPE OPTIONS
/boot  /dev/sda1 ext4   rw,relatime,seclabel
```

管理正在访问指定文件系统的进程

```bash
#lsof
lsof MOUNT_POINT		查看所有正在访问文件系统的进程

#fuser
fuser -v MOUNT_POINT	查看所有正在访问文件系统的进程
fuser -km MOUNT_POINT	终止所有正在访问文件系统的进程
```

##### 2.3.4 持久挂载

将挂载保存到 /etc/fstab 中可以下次开机时，自动启用挂载

查看帮助：

```bash
man	5 	fstab
mount	-a					#可以使新添加的挂载项生效
```

fstab格式：

```bash
[root@centos8 ~]# cat /etc/fstab 
#
UUID=ccd25378-c82e-4bea-ad12-81fba73fdf70 /                       xfs     defaults        0 0
```

- 要挂载的设备信息（UUID、LABLE等）

- 挂载点：必须是事先存在的目录

- 文件系统类型

- 挂载选项

- 转储频率：0：不做备份 1：每天转储 2：每隔一天转储

- fsck检查的文件系统的顺序：允许的数字是0 1 2

- 

  0：不自检 ，1：首先自检；一般只有rootfilesystems才用 2：非rootfilesystems使用

### 2.4 swap分区

##### 2.4.1 swap 介绍

当没有足够的 RAM 保存系统处理的数据 时会将数据写入 swap 分区，当系统缺乏 swap 空间时，内核会因 RAM 内存耗尽而终止进程。

**官方推荐推荐系统 swap 空间**

https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/installation_gui de/sect-disk-partitioning-setup-ppc#sect-recommended-partitioning-scheme-ppc

##### 2.4.2 交换分区实现过程

1.创建交换分区或者文件

```bash
[root@centos8 ~]#echo -e 'n\np\n\n\n+2G\nt\n82\nw\n' | fdisk /dev/sdc
```

2.创建swap分区

```bash
[root@centos8 ~]#mkswap /dev/sdc1
```

3.写入配置文件fstab

```bash
[root@centos8 ~]#vim /etc/fstab
UUID=d3140a7a-65b7-4cb7-8a2b-12d38aa98c6f swap         swap defaults 0 0
```

4.使用swapon -a 激活交换空间

```bash
[root@centos8 ~]#swapon -a
[root@centos8 ~]#free -h
             total       used       free     shared buff/cache   available
Mem:          3.7Gi       264Mi       3.2Gi       9.0Mi       261Mi       3.2Gi
Swap:         4.0Gi         0B       4.0Gi
[root@centos8 ~]#cat /proc/swaps
Filename Type Size Used Priority
/dev/sda5                               partition 2097148 0 -2
/dev/sdc1                               partition 2097148 0 -3
```

5.swapon和swapoff

```bash
#启用swap分区
swapon [OPTION]... [DEVICE]
	-a：激活所有的交换分区
	-p PRIORITY：指定优先级，也可在/etc/fstab 在第4列指定：pri=value
	
#禁用swap分区
swapoff [OPTION]... [DEVICE]
```

swap优先级说明：可以指定swap分区0到32767的优先级，值越大优先级越高

### 2.5 移动介质使用

##### 2.5.1 使用光盘

挂载和卸载

```bash
[root@centos8 ~]# mount /dev/cdrom /mnt/
[root@centos8 ~]# unmount	/mnt/
```

创建ISO文件

```bash
[root@centos8 ~]# cp /dev/cdrom /root/centos.iso
[root@centos8 ~]# mkisofs -r -o /root/etc.iso /etc #来自于genisoimage包
```

##### 2.5.2 USB介质

查看usb设备

```bash
[root@centos8 ~]# lsusb
[root@centos8 ~]#mount /dev/sdX# /mnt
```

查看硬件设备日志

```bash
[root@centos8 ~]#tail /var/log/messages -f
[root@centos8 ~]#dmesg
```

格式化U盘为 FAT32 文件系统

```bash
[root@centos8 ~]#dnf -y install dosfstools
[root@centos8 ~]#mkfs.vfat /dev/sdd1
mkfs.fat 4.1 (2017-01-24)
[root@centos8 ~]#mount /dev/sdd1 /mnt
```

### 2.6 磁盘管理命令

##### 2.6.1 df

df：查看文件系统磁盘的已经使用空间

格式：

```bash
df [OPTION]... [FILE]...
```

常用选项：

- -H 以10为单位
- -T 文件系统类型
- -h 以更容易阅读的单位显示
- -i inode的占用数量
- -P 以整理好的格式输出

范例：

```bash
[root@centos8 data]# df -TH
Filesystem     Type      Size  Used Avail Use% Mounted on
devtmpfs       devtmpfs  943M     0  943M   0% /dev
tmpfs          tmpfs     958M     0  958M   0% /dev/shm
tmpfs          tmpfs     958M  467k  958M   1% /run
tmpfs          tmpfs     958M     0  958M   0% /sys/fs/cgroup
/dev/vda1      xfs        54G  3.5G   51G   7% /
tmpfs          tmpfs     192M     0  192M   0% /run/user/0
[root@centos8 data]# df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
devtmpfs       devtmpfs  899M     0  899M   0% /dev
tmpfs          tmpfs     914M     0  914M   0% /dev/shm
tmpfs          tmpfs     914M  456K  914M   1% /run
tmpfs          tmpfs     914M     0  914M   0% /sys/fs/cgroup
/dev/vda1      xfs        50G  3.2G   47G   7% /
tmpfs          tmpfs     183M     0  183M   0% /run/user/0
```

##### 2.6.2 du

du：统计目录占用空间

格式

```bash
du [OPTION]... [FILE]...
```

常用选项：

- -h 以更容易阅读的单位显示
- -s 只统计文件夹的大小，而不统计子目录

##### 2.6.3 dd

dd：convert and copy a file

格式：

```bash
dd if=/PATH/FROM/SRC of=/PATH/TO/DEST bs=# count=#
```

常用选项：

- if=file 从所命名文件读取而不是从标准输入

- of=file 写到所命名的文件而不是到标准输出

- ibs=size 一次读size个byte

- obs=size 一次写size个byte

- bs=size block size, 指定块大小（既是是ibs也是obs)

- skip=blocks 从开头忽略blocks个ibs大小的块

- seek=blocks 从开头忽略blocks个obs大小的块

- count=n 复制n个bs

- conv=conversion[,conversion...] 用指定的参数转换文件

  ascii 转换 EBCDIC 为 ASCII

  ebcdic 转换 ASCII 为 EBCDIC

  lcase 把大写字符转换为小写字符

  ucase 把小写字符转换为大写字符

  nocreat 不创建输出文件

  noerror 出错时不停止

  notrunc 不截短输出文件

  sync 把每个输入块填充到ibs个字节，不足部分用空(NUL)字符补齐

  fdatasync 写完成前，物理写入输出文件

范例：

```bash
[root@centos8 data]# echo abcdef > f1.txt
[root@centos8 data]# echo 12345678 > f2.txt
[root@centos8 data]# dd if=f1.txt of=f2.txt bs=1 seek=3 
7+0 records in
7+0 records out
7 bytes copied, 6.0037e-05 s, 117 kB/s
[root@centos8 data]# cat f2.txt 
123abcdef
```

## 3.RAID

### 3.1 什么是RAID

RAID：Redundant Arrays of Inexpensive（Independent） Disks 廉价（独立）磁盘冗余阵列 1988年由加利福尼亚大学伯克利分校（University of California-Berkeley） “A Case for Redundant Arrays of Inexpensive Disks”,多个磁盘合成一个“阵列”来提供更好的性能、冗余，或者两者都提供

### 3.2 RAID级别

##### 3.2.1 RAID-0

以 chunk 单位,读写数据

组合方式：

- Disk0：A1 A3 A5 A7
- Disk0：A2 A4 A6 A8

最少磁盘数：2

##### 3.2.2 RAID-1

组合方式：

- Disk0：A1 A2 A3 A4
- Disk1：A1 A2 A3 A4

最少磁盘数：2

##### 3.2.3 RAID-4

组合方式：

- Disk0：A1 B1 C1 D1
- Disk1：A2 B2 C2 D2
- Disk2：Ap Bp Cp Dp

使用一块硬盘做校验位，至少3块硬盘才可以实现（不过校验位硬盘容易损坏）

##### 3.2.4 RAID-5

组合方式：

- Disk0：A1 B1 Cp
- Disk1：A2 Bp C2
- Disk1：Ap B3 C3

每块磁盘上都放置一个校验位，允许损坏1块磁盘，最少要3块硬盘实现

##### 3.2.5 RAID-6

组合方式：

- Disk0：A1 B1 Cp Dq
- Disk1：A1 Bp Cq D1
- Disk2：Ap Bq C2 D2
- Disk3：Aq B3 C3 Dp

每块磁盘上放置两个校验位，允许损坏2块磁盘，至少要4块硬盘实现

##### 3.2.6 RAID-10

组合方式：

- Disk0：A1 A2
- Disk1：A1 A2
- Disk2：A3 A4
- Disk3：A3 A4

先将Disk0和Disk1组成RAID1、Disk2和Disk3组成RAID1，在将两个RAID1组合成RAID0.至少要4块硬盘实现该功能。

##### 3.2.7 RAID-01

组合方式：

- Disk0：A1 A2
- Disk1：A3 A4
- Disk2：A1 A2
- Disk3：A3 A4

先将Disk0和Disk1组成RAID0、Disk2和Disk3组成RAID0，在将两个RAID1组合成RAID1.至少要4块硬盘实现该功能。

3.2.8 JBOD

组合方式：

- Disk0：A1 A2 A3
- Disk1：A4 A5 A6
- Disk2：A7 A8 A9

将多个磁盘空间合并成一个大空间使用

RAID7 可以理解为一个独立存储计算机，自身带有操作系统和管理工具，可以独立运行，理论上性能最高的 RAID模式

常用级别： RAID-0, RAID-1, RAID-5, RAID-10, RAID-50, JBOD

## 4.逻辑卷管理（LVM）

### 4.1 LVM介绍

LVM可以弹性的更改LVM的容量

**实现过程**：

- 将设备指定为物理卷
- 用一个或者多个物理卷来创建一个卷组，物理卷是用固定大小的物理区域（Physical Extent， PE）来定义的
- 创建逻辑卷，在逻辑卷上创建文件系统并挂载

### 4.2 实现逻辑卷

##### 4.2.1 pv管理工具

显示pv信息

```bash
pvs：简要pv信息显示
pvdisplay
```

删除和创建pv

```bash
pvremove /dev/DEVICE
pvcreate /dev/DEVICE
```

##### 4.2.2 vg管理工具

显示卷组信息

```bash
vgs
vgdisplay
```

创建卷组：

```bash
vgcreate	[-s|--physicalextentsize PhysicalExtentSize[bBsSkKmMgGtTpPeE]]	VolumeGroupName PhysicalDevi‐cePath [PhysicalDevicePath...]
```

扩展和减少卷组：

```bash
vgextend VolumeGroupName PhysicalDevicePath [PhysicalDevicePath...]
vgreduce VolumeGroupName PhysicalDevicePath [PhysicalDevicePath...]
```

##### 4.2.3 lv管理工具

显示逻辑卷

```bash
lvs
lvsdisplay
```

创建逻辑卷

```bash
lvcreate -L  #[mMgGtT] -n NAME VolumeGroup
#范例
lvcreate -l 60%VG -n mylv testvg
lvcreate -l 100%FREE -n yourlv testvg
```

删除逻辑卷

```bash
lvremove /dev/VG_NAME/LV_NAME
```

重设文件系统大小，用于新增逻辑卷后扩展旧文件系统大小

```bash
fsadm [options] resize device [new_size[BKMGTEP]]
#调整ext系列文件系统大小
resize2fs [-f] [-F] [-M] [-P] [-p] device [new_size]
#调整xfs系列文件系统大小
xfs_growfs /mountpoints
```

##### 4.2.4 扩展和缩减逻辑卷

注意：缩减有数据损坏的风险，建议先备份再缩减，xfs文件系统不支持缩减

```bash
#1.扩展逻辑卷
lvextend -L [+]#[mMgGtT] /dev/VG_NAME/LV_NAME
#针对ext
resize2fs /dev/VG_NAME/LV_NAME
#针对xfs
xfs_growfs MOUNTPOINT
lvresize -r -l +100%FREE /dev/VG_NAME/LV_NAME

#2.缩减逻辑卷
umount /dev/VG_NAME/LV_NAME
e2fsck -f /dev/VG_NAME/LV_NAME
resize2fs /dev/VG_NAME/LV_NAME #[mMgGtT]
lvreduce -L [-]#[mMgGtT] /dev/VG_NAME/LV_NAME
mount
```

### 4.3 逻辑卷快照

##### 4.3.1 逻辑卷快照原理

快照就是将当时的系统信息记录下来，就好像照相一般，若将来有任何数据改动了，则原始数据会被移 动到快照区，没有改动的区域则由快照区和文件系统共享

快照特点：

- 备份速度快，瞬间完
- 应用场景是测试环境，不能完成代替备份
- 快照后，逻辑卷的修改速度会一定有影响

##### 4.3.2 实现逻辑卷快照

范例：

```bash
#为现有逻辑卷创建快照
lvcreate -l 64 -s -n data-snapshot -p r /dev/vg0/data
#挂载快照
mkdir  -p /mnt/snap
mount -o ro /dev/vg0/data-snapshot   /mnt/snap
#恢复快照
umount /dev/vg0/data-snapshot
umount /dev/vg0/data
lvconvert --merge /dev/vg0/data-snapshot
#删除快照
umount /mnt/databackup
lvremove /dev/vg0/databackup
```

练习：

1.创建一个2G的文件系统，块大小为2048byte,预留1%的可用空间，文件系统ext4,卷标为TEST,要求此分区开机后自动挂载至/test目录，并且有acl挂载选项。

```bash
[root@centos8 ~]# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): n
Partition number (2-128, default 2): 2
First sector (2099200-41943006, default 2099200): 
Last sector, +sectors or +size{K,M,G,T,P} (2099200-41943006, default 41943006): +2G

Created a new partition 2 of type 'Linux filesystem' and of size 2 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

[root@centos8 ~]# mkfs.ext4 -L TEST -m 1 -b 2048 /dev/sdb2
mke2fs 1.45.4 (23-Sep-2019)
Creating filesystem with 524288 4k blocks and 131072 inodes
Filesystem UUID: 49dcc4bf-f2c2-4ef0-b707-bbc1f3081d7a
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done

[root@centos8 ~]# blkid /dev/sdb2
/dev/sdb2: LABEL="TEST" UUID="49dcc4bf-f2c2-4ef0-b707-bbc1f3081d7a" TYPE="ext4" PARTUUID="f88db445-edbd-0e43-8453-0f70b1fc198a"

#写入配置文件实现持久挂载
[root@centos8 ~]# vim /etc/fstab
UUID=ccd25378-c82e-4bea-ad12-81fba73fdf70 /                       xfs     defaults        0 0
UUID=49dcc4bf-f2c2-4ef0-b707-bbc1f3081d7a /test                   xfs     defaults,acl    0 0 
[root@centos8 ~]# mount -a
```

3.创建一个至少有两个PV组成的大小为20G的名为testvg的VG;要求PE大小为16MB,而后在卷组中创建大小为5G的逻辑卷testlv；挂载至/data/test目录

```bash
#1.添加两块新硬盘sdb和sdc，使用fdisk创建sdb1和sdc1并且将分区类型改为8e
[root@Centos7 data]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  200G  0 disk 
├─sda1   8:1    0    1G  0 part /boot
├─sda2   8:2    0  100G  0 part /
├─sda3   8:3    0   50G  0 part /data
├─sda4   8:4    0    1K  0 part 
└─sda5   8:5    0    4G  0 part [SWAP]
sdb      8:16   0   20G  0 disk 
└─sdb1   8:17   0   10G  0 part 
sdc      8:32   0   20G  0 disk 
sr0     11:0    1  9.5G  0 rom  /mnt

[root@Centos7 data]# fdisk -l

Disk /dev/sdb: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xcf597f02

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048    20973567    10485760   8e  Linux LVM

Disk /dev/sdc: 21.5 GB, 21474836480 bytes, 41943040 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x5cec7bc0

   Device Boot      Start         End      Blocks   Id  System
/dev/sdc1            2048    20973567    10485760   8e  Linux LVM

#2.创建pv
[root@Centos7 data]# pvcreate /dev/sdb1 /dev/sdc1
  Physical volume "/dev/sdb1" successfully created.
  Physical volume "/dev/sdc1" successfully created.

#3.创建物理卷组，设置PE大小为16M
[root@Centos7 data]# vgcreate -s 16M testvg /dev/sdb1 /dev/sdc1
  Volume group "testvg" successfully created
[root@Centos7 data]# vgdisplay
  --- Volume group ---
  VG Name               testvg
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               <19.97 GiB
  PE Size               16.00 MiB
  Total PE              1278
  Alloc PE / Size       0 / 0   
  Free  PE / Size       1278 / <19.97 GiB
  VG UUID               H6l3Hu-WKI7-zAIo-A4Zy-k3hp-ojkP-YEqI1H

 #4.创建逻辑卷
[root@Centos7 data]# lvcreate -L 5G -n testlv testvg
  Logical volume "testlv" created.
 [root@Centos7 data]# lvs
  LV     VG     Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  testlv testvg -wi-a----- 5.00g                                      
[root@Centos7 data]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/testvg/testlv
  LV Name                testlv
  VG Name                testvg
  LV UUID                YCrkZc-J2PL-PwZF-DWvu-8xvO-EuQm-xo2RIk
  LV Write Access        read/write
  LV Creation host, time Centos7, 2021-02-04 19:56:33 +0800
  LV Status              available
  # open                 0
  LV Size                5.00 GiB
  Current LE             320
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0
 
#5.创建文件系统并挂载至挂载点
[root@Centos7 data]# mkfs.ext4 /dev/testvg/testlv 
[root@Centos7 data]# blkid
/dev/sdb1: UUID="8Ru2tC-P9rW-3TDt-fHiH-BEDT-32sq-4rt9OK" TYPE="LVM2_member" 
/dev/sdc1: UUID="vZDdu2-1obH-hLfi-NkZo-LCad-UJAU-ZlWLR7" TYPE="LVM2_member" 
/dev/mapper/testvg-testlv: UUID="682fa042-9b2e-4376-8ef9-1c785fc13bca" TYPE="ext4"
[root@Centos7 data]# vim /etc/fstab 
UUID=682fa042-9b2e-4376-8ef9-1c785fc13bca /data/test              ext4    defaults        0 0
[root@Centos7 data]# mount -a
[root@Centos7 data]# df -h
Filesystem                 Size  Used Avail Use% Mounted on
devtmpfs                   900M     0  900M   0% /dev
tmpfs                      910M     0  910M   0% /dev/shm
tmpfs                      910M  9.6M  901M   2% /run
tmpfs                      910M     0  910M   0% /sys/fs/cgroup
/dev/sda2                  100G  1.5G   99G   2% /
/dev/sda3                   50G   33M   50G   1% /data
/dev/sda1                 1014M  142M  873M  14% /boot
tmpfs                      182M     0  182M   0% /run/user/0
/dev/sr0                   9.5G  9.5G     0 100% /mnt
/dev/mapper/testvg-testlv  4.8G   20M  4.6G   1% /data/test

#6.为逻辑卷扩展5G空间
[root@Centos7 data]# lvextend -L +5G /dev/testvg/testlv 
  Size of logical volume testvg/testlv changed from 5.00 GiB (320 extents) to 10.00 GiB (640 extents).
  Logical volume testvg/testlv successfully resized.
[root@Centos7 data]# resize2fs /dev/testvg/testlv 
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/testvg/testlv is mounted on /data/test; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 2
The filesystem on /dev/testvg/testlv is now 2621440 blocks long.

[root@Centos7 data]# df -h
Filesystem                 Size  Used Avail Use% Mounted on
devtmpfs                   900M     0  900M   0% /dev
tmpfs                      910M     0  910M   0% /dev/shm
tmpfs                      910M  9.6M  901M   2% /run
tmpfs                      910M     0  910M   0% /sys/fs/cgroup
/dev/sda2                  100G  1.5G   99G   2% /
/dev/sda3                   50G   33M   50G   1% /data
/dev/sda1                 1014M  142M  873M  14% /boot
tmpfs                      182M     0  182M   0% /run/user/0
/dev/sr0                   9.5G  9.5G     0 100% /mnt
/dev/mapper/testvg-testlv  9.8G   23M  9.3G   1% /data/test
```

# 2.网络管理和配置

## 1.计算机网络体系结构和参考模型

### 1.1 计算机网络分层结构及基本概念

**层次划分的必要性：**

两个系统中计算机的通信是一个很复杂的过程，为了降低协议设计和调试过程的复杂性，也为了便于对网络进行研究、实现和维护，促进标准化工作，通常对计算机的网络体系结构以分层的方式进行建模。

**分层的基本原则：**

- 每层都实现一种相对独立的功能，降低大系统的复杂度
- 各层之间界面自然清晰，易于理解，互相交流尽可能少
- 各层功能精确定义独立于具体实现的方法，可以采用最合适的技术来实现
- 保持下层对上层的独立性，上层单向使用下层提供的服务
- 整个分层结构应能促进标准化工作

**协议：**交换数据规则的集合，它是平行的。

**接口：**同一结点内相邻两层间交换信息的连接点，每层只能为紧邻的层次之间定义接口

**服务：**下层为紧邻的上层提供的功能调用，它是垂直的。

### 1.2 OSI参考模型

国际标准化组织（OSI）提出的网络体系结构模型，称为开放系统互连参考模型，通常简称为OSI参考模型。OSI的层次结构和通信过程如图所示。
[![1](https://myblognote.oss-cn-beijing.aliyuncs.com/img/20210209165901.png)](https://myblognote.oss-cn-beijing.aliyuncs.com/img/20210209165901.png)

- 第一层 物理层

  **物理传输层的单位是比特**，任务是透明的传输比特流，它负责管理电脑通信设备和网 络媒体之间的互通。包括了针脚、电压、线缆规范、集线器、中继器、网卡、主机接口卡等

- 第二层 数据链路层

  **数据链路层的传输单位是帧**，任务是将网络层传来的IP数据报组成帧。功能可以概括为成帧、差错控制、流量控制和传输管理等，实现点到点的通信。

- 第三层 网络层

  **网络层的传输单位是数据报**，主要任务是吧网络层的协议数据单元（分组）从源端传到目的端，为分组交换网上的不同主机提供通信服务。关键是对分组进行路由选择，并实现流量控制、拥塞控制、差错控制和网际互连等功能

- 第四层 传输层

  **传输层的传输单位是报文段（TCP）或用户数据报（UDP）**,传输层负责主机中两个进程之间的通信。

- 第五层 会话层

  管理主机间的会话进程，包括建立、管理及终止进程间的会话

- 第六层 表示层

  数据格式交换、数据加密解密和数据压缩恢复。

- 第七层 应用层

  应用层（Application Layer）提供为应用软件而设的接口，以设置与另一应用软件之间的通信。例如: HTTP、HTTPS、FTP、TELNET、SSH、SMTP、POP3、MySQL等

在高层次中如：会话层、表示层和应用层的协议数据单元 （PDU：Protocol Data Unit）都是消息 message

范例：

```bash
#查看网卡是否启用
[root@centos8 data]# mii-tool -v eth0
SIOCGMIIPHY on 'eth0' failed: Operation not supported
[root@centos8 data]# ethtool eth0
Settings for eth0:
	Supported ports: [ ]
	Supported link modes:   Not reported
	Supported pause frame use: No
	Supports auto-negotiation: No
	Supported FEC modes: Not reported
	Advertised link modes:  Not reported
	Advertised pause frame use: No
	Advertised auto-negotiation: No
	Advertised FEC modes: Not reported
	Speed: Unknown!
	Duplex: Unknown! (255)
	Port: Other
	PHYAD: 0
	Transceiver: internal
	Auto-negotiation: off
	Link detected: yes
[root@centos8 data]# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 00:16:3e:12:a3:c2 brd ff:ff:ff:ff:ff:ff
```

### 1.3 TCP/IP模型

OSI参考模型与TCP/IP参考模型如图：
[![2](https://myblognote.oss-cn-beijing.aliyuncs.com/img/20210209183614.png)](https://myblognote.oss-cn-beijing.aliyuncs.com/img/20210209183614.png)

- 面向连接

  分为三个阶段：1.建立连接；2.传输数据；3.释放连接

- 面向无连接

  直接进行数据传输

- OSI先有模型；TCP/IP先有协议，后有模型

5层参考模型：综合了OSI和TCP/IP的优点[![3](https://myblognote.oss-cn-beijing.aliyuncs.com/img/20210209184733.png)](https://myblognote.oss-cn-beijing.aliyuncs.com/img/20210209184733.png)

## 2.传输层协议

### 2.1 传输层介绍

##### 2.1.1传输层功能

- 提供进程与进程之间的逻辑通信

- 复用和分用

  复用：应用层所有的应用进程都可以通过传输层再传输到网络层

  分用：传输层从网络层接收到数据后交付指明的应用进程

- 对收到的报文进行差错检测

- 传输层的两种协议

##### 2.1.2 传输层的寻址与端口

端口是传输层的接口，用于标识主机中的应用进程

端口长度为16bit,能表示65536个不同的端口号

- 0-1023：系统端口或特权端口(仅管理员可用) ，众所周知，永久的分配给固定的系统应用使用， 22/tcp(ssh), 80/tcp(http), 443/tcp(https)
- 1024-49151：用户端口或注册端口，但要求并不严格，分配给程序注册为某应用使用， 1433/tcp(SqlServer), 1521/tcp(oracle),3306/tcp(mysql),11211/tcp/udp (memcached)
- 49152-65535：动态或私有端口，客户端随机使用端口，范围定 义：/proc/sys/net/ipv4/ip_local_port_range

**套接字Socket=主机IP地址+端口号**

范例：

```bash
#调整客户端的动态端口范围
[root@centos8 data]# cat /proc/sys/net/ipv4/ip_local_port_range 
32768	60999
[root@centos8 data]# echo 20000 62000 > /proc/sys/net/ipv4/ip_local_port_range
[root@centos8 data]# cat /proc/sys/net/ipv4/ip_local_port_range 
20000 62000

#找到端口冲突的应用程序
[root@centos8 data]# nc -l 22
Ncat: bind to 0.0.0.0:22: Address already in use. QUITTING.
[root@centos8 data]# ss -ntlp
State   Recv-Q  Send-Q   Local Address:Port   Peer Address:Port                                                                
LISTEN  0       128            0.0.0.0:5355        0.0.0.0:*      users:(("systemd-resolve",pid=910,fd=13))                    
LISTEN  0       128            0.0.0.0:80          0.0.0.0:*      users:(("nginx",pid=133609,fd=8),("nginx",pid=133608,fd=8))  
LISTEN  0       128            0.0.0.0:22          0.0.0.0:*      users:(("sshd",pid=976,fd=5))                                
LISTEN  0       100          127.0.0.1:25          0.0.0.0:*      users:(("master",pid=113712,fd=16))                          
LISTEN  0       128               [::]:5355           [::]:*      users:(("systemd-resolve",pid=910,fd=15))                    
LISTEN  0       128               [::]:80             [::]:*      users:(("nginx",pid=133609,fd=9),("nginx",pid=133608,fd=9))  
LISTEN  0       100              [::1]:25             [::]:*      users:(("master",pid=113712,fd=17))
[root@centos8 data]# lsof -i :22
COMMAND    PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
sshd       976 root    5u  IPv4   24095      0t0  TCP *:ssh (LISTEN)
sshd    261372 root    5u  IPv4 3935567      0t0  TCP centos8.2:ssh->49.118.65.208:7218 (ESTABLISHED)
sshd    261386 root    5u  IPv4 3935567      0t0  TCP centos8.2:ssh->49.118.65.208:7218 (ESTABLISHED)

#判断端口是否正在打开
[root@centos8 data]# < /dev/tcp/127.0.0.1/80
[root@centos8 data]# echo $?
0
[root@centos8 data]# < /dev/tcp/127.0.0.1/8090
-bash: connect: Connection refused
-bash: /dev/tcp/127.0.0.1/8090: Connection refused
[root@centos8 data]# echo $?
1
```

### 2.2 TCP协议

##### 2.2.1 TCP协议特性

- TCP是面向连接的传输层协议

- 每一条TCP连接只能有两个端点，每一条TCP连接只能是点对点

- TCP提供可靠的服务

- TCP提供全双工通信。

  发送缓存：准备发送的数据和已发送但尚未收到确认的数据

  接收缓存：按序到达但尚未被接受应用程序的读取的数据和不按序到达的数据

- TCP面向字节流：TCP把应用程序交下来的数据看成仅仅是一连串的无结构的字节流

[![1](https://myblognote.oss-cn-beijing.aliyuncs.com/linux%E5%9F%BA%E7%A1%80%E5%8F%8A%E7%B3%BB%E7%BB%9F%E5%AE%89%E8%A3%85/1.png)](https://myblognote.oss-cn-beijing.aliyuncs.com/linux基础及系统安装/1.png)

##### 2.2.2 TCP报文段首部格式

[![2](https://myblognote.oss-cn-beijing.aliyuncs.com/linux%E5%9F%BA%E7%A1%80%E5%8F%8A%E7%B3%BB%E7%BB%9F%E5%AE%89%E8%A3%85/2.png)](https://myblognote.oss-cn-beijing.aliyuncs.com/linux基础及系统安装/2.png)

- URG：URG=1时，标明此报文段中有紧急数据，是高优先级的数据，应尽快传送，不用在缓存里排队，配合紧急指针字段使用。
- ACK：表示是否前面确认号字段是否有效。只有当ACK=1时，前面的确认号字段才有效。TCP规 定，连接建立后，ACK必须为1,带ACK标志的TCP报文段称为确认报文段
- PSH：提示接收端应用程序应该立即从TCP接收缓冲区中读走数据，为接收后续数据腾出空间。如 果为1，则表示对方应当立即把数据提交给上层应用，而不是缓存起来。
- 复位RST：RST=1时，表明TCP连接中出现严重差错，必须释放连接，然后再重新建立传输链接。
- 同步位SYN：SYN=1时，表明是一个连接请求/连接接受报文。
- 终止位FIN：FIN=1时，表明此报文段发送方数据已发完，要求释放连接。
- 窗口：指的是发送本报文段的一方的接收窗口，即现在允许对方发送的数据量。
- 检验和：检验首部+数据，检验时要加上12B伪首部，第四个字段为6。
- 紧急指针：URG=1时才有意义，指出本报文段中紧急数据的字节数。
- 选项：最大报文段长度MSS、窗口扩大、时间戳、选择确认…

##### 2.2.3 TCP的连接管理

TCP连接传输三个阶段：

 连接建立→数据传送→连接释放

**TCP连接建立：**[![1](https://myblognote.oss-cn-beijing.aliyuncs.com/img/20210211192829.png)](https://myblognote.oss-cn-beijing.aliyuncs.com/img/20210211192829.png)

seq为序号段数值为随机，ACK与ack应是同时存在，ack为确认号段。

**TCP连接释放：**

[![2](https://myblognote.oss-cn-beijing.aliyuncs.com/img/20210211220747.png)](https://myblognote.oss-cn-beijing.aliyuncs.com/img/20210211220747.png)

**内核TCP参数优化**

参看帮助： 使用man tcp 、也可以访问官网查看 /proc/sys/net目录中对应网络相关文档https://man7.org/linux/man-pages/man5/proc.5.html

### 2.3 UDP协议

##### 2.3.1 UDP协议特性

无连接，减少开销和发送数据之前的时延

不保证可靠交付

面向报文，适合一次性传输少量数据的网络应用

无拥塞控制

首部开销小，8B，TCP20B

2.3.2 UDP首部格式

[![3](https://myblognote.oss-cn-beijing.aliyuncs.com/img/20210211230835.png)](https://myblognote.oss-cn-beijing.aliyuncs.com/img/20210211230835.png)

## 3.网络层

### 3.5 IP地址

##### 3.5.1 IP地址组成

IP地址：全世界唯一的32位/4字节标识符，标识路由器主机的接口。

IP地址={<网络号>,<主机号>}

##### 3.5.2 IP地址分类[![1](https://myblognote.oss-cn-beijing.aliyuncs.com/img/20210218221016.png)](https://myblognote.oss-cn-beijing.aliyuncs.com/img/20210218221016.png)

**私有IP地址**：

- A类：10.0.0.0~10.255.255.255
- B类：172.16.0.0~172.31.255.255
- C类：192.168.0.0~192.168.255.255

##### 3.5.3 特殊地址

- 0.0.0.0

  0.0.0.0不是一个真正意义上的IP地址。它表示所有不清楚的主机和目的网络

- 255.255.255.255

  限制广播地址。对本机来说，这个地址指本网段内(同一广播域)的所有主机

- 127.0.0.1～127.255.255.254

  本机回环地址，主要用于测试。在传输介质上永远不应该出现目的地址为“127.0.0.1”的 数据包

- 224.0.0.0到239.255.255.255

  组播地址，224.0.0.1特指所有主机，224.0.0.2特指所有路由器。224.0.0.5指OSPF 路由器，地址 多用于一些特定的程序以及多媒体程序

- 169.254.x.x

  如果Windows主机使用了DHCP自动分配IP地址，而又无法从DHCP服务器获取地址，系统会为主 机分配这样地址

[![2](https://myblognote.oss-cn-beijing.aliyuncs.com/img/20210218224740.png)](https://myblognote.oss-cn-beijing.aliyuncs.com/img/20210218224740.png)

##### 3.5.5 子网掩码

无分类域间路由选择CIDR：

1.消除了传统的A类，B类和C类地址以及划分子网的概念。

 CIDR：无类域间路由，目前的网络已不再按A，B，C类划分网段，可以任意指定网段的范围

 CIDR记法：IP地址后加上“/”，然后写上网络前缀（可以任意长度）的位数。 e.g. 128.14.32.0/20

2.融合子网地址与子网掩码，方便子网划分。

 CIDR把网络前缀都相同的连续的IP地址组成一个 “CIDR地址块”。

范例：

一个主机：172.16.1.100/28

1、此主机所在的网段最多有多少主机?主机数=2^(32-28)-2=14

2、网络ID? IP和子网掩码相与，172.16.1.96

- 10101100 00010000 00000001 01100100

- 11111111 11111111 11111111 11110000

- 10101100 00010000 00000000 01100000

   172 16 1 96

3、此网段的主机中最小的IP：172.16.1.97，最大的IP?172.16.1.110

- 10101100 00010000 00000000 01100001
- 10101100 00010000 00000000 01111110

##### 3.5.6 子网划分和超网划分

划分子网：将一个大的网络（主机数多）划分成多个小的网络（主机数少），主机ID位数变少，网络ID 位数变多，网络ID位向主机ID位借位

合并超网：将多个小网络合并成一个大网，主机ID位向网络ID位借位

范例：

```bash
中国电信10.0.0.0/8 给32个各省公司划分对应的子网
1）每个省公司的子网的netmask？
2^5>=32 借5位网络ID
8+5=13
255.248.0.0

2)每个省公司的子网的主机数有多少？
2^(32-13)-2=524286

3）河南省得到第10个子网，网络ID？
10.00000 000.0.0/13
10.01001 000.0.0/13
10.72.0.0/13

4）河南省得到第10个子网的最小IP和最大的IP？
10.01001 000.0.1
10.01001 111.11111111.11111110
10.72.0.1---10.79.255.254

5）所有子网中最大，最小的子网的netid？
10.00000 000.0.0/13 10.0.0.0/13
10.11111 000.0.0/13 10.248.0.0/13
```

## 4.网络配置

### 4.1 网络配置命令

##### 4.1.1 ip命令

来自于iproute包，可用于代替ifconfig

格式：

```bash
ip [ OPTIONS ] OBJECT { COMMAND | help }
OBJECT := { link | addr | route }
ip link - network device configuration
set dev IFACE，可设置属性：up and down：激活或禁用指定接口，相当于 ifup/ifdown
show [dev IFACE] [up]：：指定接口 ，up 仅显示处于激活状态的接口
```

ip 地址管理

```bash
ip addr { add | del } IFADDR dev STRING [label LABEL] [scope {global|link|host}]
[broadcast ADDRESS]

[label LABEL]：添加地址时指明网卡别名
[scope {global|link|host}]：指明作用域,global: 全局可用.link: 仅链接可用,host: 本机可
用
[broadcast ADDRESS]：指明广播地址

ip address show
ip addr flush 
```

范例：

```bash
#禁用网卡
ip link set eth1 down

#网卡改名
ip link set eth1 name wangnet  

#启用网卡
ip link set wangnet up

#网卡别名
ip addr add 172.16.100.100/16 dev eth0 label eth0:0
ip addr del 172.16.100.100/16 dev eth0 label eth0:0

#清除网络地址
ip addr flush dev eth0 
```

##### 4.1.2 ss 命令

来自于iproute包，代替netstat，netstat 通过遍历 /proc来获取 socket信息，ss 使用 netlink与内核 tcp_diag 模块通信获取 socket 信息

格式：

```bash
ss [OPTION]... [FILTER]
```

选项：

- -t: tcp协议相关
- -u: udp协议相关
- -w: 裸套接字相关
- -x：unix sock相关
- -l: listen状态的连接
- -a: 所有
- -n: 数字格式
- -p: 相关的程序及PID
- -e: 扩展的信息
- -m：内存用量
- -o：计时器信息

范例：

```bash
#显示本地打开的所有端口
ss -l

#显示每个进程具体打开的socket
ss -pl

#显示所有tcp socket
ss -t -a

#显示所有的UDP Socekt
ss -u -a

#显示所有已建立的ssh连接
ss -o state established '( dport = :ssh or sport = :ssh )'

#显示所有已建立的HTTP连接
ss -o state established '( dport = :http or sport = :http )'
[root@centos8 ~]#ss -no state established '( dport = :21 or sport = :21 )'
Netid               Recv-Q               Send-Q                                
  Local Address:Port                                   Peer Address:Port      
         
tcp                 0                     0                                    
[::ffff:10.0.0.8]:21                                 [::ffff:10.0.0.7]:46638    
            timer:(keepalive,119min,0)

#列出当前socket详细信息
ss -s
```

### 4.2 网络配置文件

##### 4.2.1 网络基本配置文件

IP、MASK、GW、DNS相关的配置文件：

```bash
/etc/sysconfig/network-scripts/ifcfg-IFACE
/usr/share/doc/initcripts-*/sysconfig.txt
```

##### 4.2.2 配置当前主机的主机名

```bash
#centos6 之前版本
/etc/sysconfig/network
HOSTNAME=

#centos7 以后版
/etc/hostname
HOSTNAME
```

##### 4.2.3 路由相关的配置文件

```bash
/etc/sysconfig/network-scripts/route-IFACE
两种风格：
(1) TARGET via GW
如：10.0.0.0/8 via 172.16.0.1

(2) 每三行定义一条路由
ADDRESS#=TARGET
NETMASK#=mask
GATEWAY#=GW
```

### 4.3 多网卡 bonding

将多块网卡绑定同一IP地址对外提供服务，可以实现高可用或者负载均衡。直接给两块网卡设置同一IP 地址是不可以的。通过 bonding，虚拟一块网卡对外提供连接，物理网卡的被修改为相同的MAC地址

##### 4.3.1 nmcli实现bonding

```bash
#创建名位bond0的绑定
[root@centos8 ~]# nmcli con add type bond con-name bond0 ifname bond0 mode active-backup
Connection 'mybond0' (9301ff97-abbc-4432-aad1-246d7faea7fb) successfully added.

#添加从属接口
[root@centos8 ~]#nmcli con add type bond-slave ifname ens160 master bond0
[root@centos8 ~]#nmcli con add type bond-slave ifname ens192 master bond0

#先启动从属接口
[root@centos8 ~]# nmcli con up bond-slave-ens160
[root@centos8 ~]# nmcli con up bond-slave-ens192

#启动绑定网卡
[root@centos8 ~]# nmcli con up bond0

#生成的相关配置文件
[root@centos8 ~]# ls /etc/sysconfig/network-scripts/
ifcfg-bond0-1  ifcfg-bond-slave-ens160  ifcfg-bond-slave-ens192

#注意：本次是在Vmware虚拟机中进行，还需将ifcfg-bond0-1配置文件中fail_over_mac设定为1，若使用默认选项0，由于mac地址相同，会出现断开一个网卡无法，另一个网卡不能发挥作用情况
[root@centos8 ~]# cat !*ifcfg-bond0-1
cat /etc/sysconfig/network-scripts/ifcfg-bond0-1
BONDING_OPTS="mode=active-backup fail_over_mac=1"
...
```

## 5 Ubuntu网络配置

ubuntu在线帮助文档（部分版本支持中文）http://manpages.ubuntu.com/manpages/

ubuntu：focal版本中文帮助文档http://manpages.ubuntu.com/manpages/focal/zh_CN/

### 5.1主机名

hostnamectl

```bash
hostnamectl [OPTIONS...] {COMMAND}
```

设置主机名

```bash
root@ubuntu:~# cat /etc/hostname 
ubuntu
root@ubuntu:~# hostnamectl set-hostname ubuntu20
root@ubuntu:~# exit
logout
user1@ubuntu:~$ su - root
Password: 
root@ubuntu20:~# 
```

### 5.2 网卡名称

默认ubuntu的网卡名称和 CentOS 7 类似

设备节点的名字，可以通过"`/etc/udev/rules.d/`"里的 udev 文件来配置，规则设置可以参考/usr/share/doc/udev/writing_udev_rules/index.html

更多详细信息可以查看https://www.debian.org/doc/manuals/debian-reference/index.zh-cn.html中文在线文档

```bash
#修改配置文件为下面形式
root@ubuntu1804:~#vi /etc/default/grub
GRUB_CMDLINE_LINUX="net.ifnames=0"
#或者sed修改
root@ubuntu1804:~# sed -i.bak '/^GRUB_CMDLINE_LINUX=/s#"$#net.ifnames=0"#'
/etc/default/grub
#生效新的grub.cfg文件
root@ubuntu1804:~# grub-mkconfig -o /boot/grub/grub.cfg
#或者
root@ubuntu1804:~# update-grub
root@ubuntu1804:~# grep net.ifnames /boot/grub/grub.cfg
       linux /vmlinuz-4.15.0-96-generic root=UUID=51517b88-7e2b-4d4a-8c14-
fe1a48ba153c ro net.ifnames=0
       linux /vmlinuz-4.15.0-96-generic root=UUID=51517b88-7e2b-4d4a8c14-fe1a48ba153c
ro net.ifnames=0
       linux /vmlinuz-4.15.0-96-generic root=UUID=51517b88-7e2b-4d4a8c14-fe1a48ba153c
ro recovery nomodeset net.ifnames=0
       linux /vmlinuz-4.15.0-76-generic root=UUID=51517b88-7e2b-4d4a8c14-fe1a48ba153c
ro net.ifnames=0
       linux /vmlinuz-4.15.0-76-generic root=UUID=51517b88-7e2b-4d4a8c14-fe1a48ba153c
ro recovery nomodeset net.ifnames=0

#重启生效
root@ubuntu1804:~# reboot 
```

### 5.3 Ubuntu网卡配置

https://ubuntu.com/server/docs/network-configuration

##### 5.3.1 配置自动获取IP

```bash
root@ubuntu20:~# man 5 netplan
root@ubuntu20:~# cat /etc/netplan/00-installer-config.yaml 
# This is the network config written by 'subiquity'
network:
  ethernets:
    ens33:
      dhcp4: true
  version: 2
  
#使用netplan命令使配置生效
root@ubuntu20:~# netplan apply
```

##### 5.2.2 配置静态IP

```bash
root@ubuntu1804:~#vim /etc/netplan/01-netcfg.yaml
network:
 version: 2
 renderer: networkd
 ethernets:
   eth0:
     addresses: [192.168.8.10/24,10.0.0.10/8]  #或者用下面两行,两种格式不能混用
  - 192.168.8.10/24
      - 10.0.0.10/8
     gateway4: 192.168.8.1
     nameservers:
   search: [google.com]
   addresses: [180.76.76.76, 8.8.8.8, 1.1.1.1]
```

# 3.进程管理和计划任务

## 1 进程介绍

### 1.1 进程概念

Process: 运行中的程序的一个副本，是被载入内存的一个指令集合，是资源分配的单位

### 1.2 系统进程状态

Linux系统进程的状态：

- 运行态：running
- 就绪态：ready
- 睡眠态：分为两种，可中断：interruptable，不可中断：uninterruptable
- 停止态：stopped，暂停于内存，但不会被调度，除非手动启动
- 僵尸态：zombie，僵尸态，结束进程，父进程结束前，子进程不关闭，杀死父进程可以关闭僵死 态 的子进程

### 1.3 进程优先级

普通进程的静态优先级：

- nice值：：-20到19，对应系统优先级100-139或99

实时优先级（动态优先级）：

- 99-0 值最大优先级最高

## 2 进程管理工具

### 2.1 pstree

查看进程树

格式：

```bash
 pstree   [OPTION] [ PID | USER ]
```

常用选项：

- -p 显示PID
- -T不显示线程数量,默认显示线程

范例：

```bash
[root@centos8 ~]$pstree -p user
bash(446775)───ping(446800)
```

### 2.2 ps

ps 即process state，可以进程当前状态的快照，默认显示当前终端中的进程，Linux系统各进程的相关 信息均保存在/proc/PID目录下的各文件中

格式：

```bash
ps [options]
```

常用选项：

BSD风格选项

- a 选项包括所有终端（tty）中的进程
- x 所有无终端（tty）的进程
- u 显示进程所有者的信息
- f 显示进程树
- k|--sort 对属性排序,属性前加 - 表示倒序
- o 属性... 选项显示定制的属性信息：pid、cmd、%cpu

UNIX风格选项

- -e 显示所有进程
- -f 显示完整格式程序信息
- -u 指定有效的用户ID或名称

GUN

- --help 查看帮助
- --sort 对属性排序相当于BSD k选项

ps输出属性

```bash
[root@centos8 ~]$ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root          22  0.0  0.0      0     0 ?        SN    2020   0:00 [ksmd]

[root@centos8 ~]$ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root          10       2  0  2020 ?        00:02:42 [rcu_sched]

C : ps -ef 显示列 C 表示cpu利用率
VSZ: Virtual memory SiZe，虚拟内存集，线性内存
RSS: ReSident Size, 常驻内存集,即真正占用的内存
STAT: 进程状态
	R：running
 	S: interruptable sleeping
 	D: uninterruptable sleeping
 	T: stopped
	Z: zombie
 	+: 前台进程
 	l: 多线程进程
	L：内存分页并带锁
	N：低优先级进程
	<: 高优先级进程
	s: session leader，会话（子进程）发起者
 	I：Idle kernel thread，CentOS 8 新特性
ni: nice值
pri: priority 优先级
rtprio: 实时优先级
psr: processor CPU编号
```

常用组合：

```bash
aux	相比-ef该选项显示cpu%值更详细
-ef	
-eFH
```

范例：

```bash
[root@centos8 ~]$ps auxf
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           2  0.0  0.0      0     0 ?        S     2020   0:01 [kthreadd]
root           3  0.0  0.0      0     0 ?        I<    2020   0:00  \_ [rcu_gp]
```

按CPU利用率倒序排序centons6以下版本不支持

```bash
[root@centos8 ~]$ps axo pid,cmd,%cpu,%mem k -%cpu
    PID CMD                         %CPU %MEM
 334445 /usr/local/aegis/aegis_clie  0.5  3.1
      1 /usr/lib/systemd/systemd --  0.0  0.6
      2 [kthreadd]                   0.0  0.0
      3 [rcu_gp]                     0.0  0.0
```

范例：实现进程和CPU的绑定

```bash
[root@centos8 ~]#taskset --help
Usage: taskset [options] [mask | cpu-list] [pid|cmd [args...]]
```

### 2.3 prtstat

显示详细的进程信息

格式：

```bash
prtstat [options] PID ...
```

选项：

-r raw格式显示

```bash
[root@centos8 ~]$prtstat -r 446775
         pid: 446775         	                  comm: bash
       state: S              	                  ppid: 446774
        pgrp: 446775         	               session: 446742
      tty_nr: 34817          	                 tpgid: 446852
       flags: 400100         	                minflt: 1320
     cminflt: 2894           	                majflt: 0
     cmajflt: 0              	                 utime: 1
       stime: 0              	                cutime: 0
      cstime: 15             	              priority: 20
        nice: 0              	           num_threads: 1
 itrealvalue: 0              	             starttime: 1017091513
       vsize: 24870912       	                   rss: 1245
      rsslim: 18446744073709551615	             startcode: 94494540193792
     endcode: 94494541273016 	            startstack: 140731806454160
     kstkesp: 0              	               kstkeip: 0
       wchan: 0              	                 nswap: 0
      cnswap: 0              	           exit_signal: 17
   processor: 0              	           rt_priority: 0
      policy: 0              	 delayaccr_blkio_ticks: 0
  guest_time: 0              	           cguest_time: 0
```

### 2.4 nice和renice

nice

以指定的优先级来启动进程

格式：

```bash
nice [OPTION] [COMMAND [ARG]...]
```

选项：

- -n 指定优先级值，默认值为10

renice

调整正在执行进程的优先级

格式：

```bash
renice [-n] priority pid...
```

范例：

```bash
[root@centos8 ~]$nice -n -20 ping 127.1.1.1
[root@centos8 ~]$ps axo pid,cmd,nice | grep ping | grep -v grep
 447048 ping 127.1.1.1              -20
```

### 2.5 pgrep

按照条件搜索进程，也可以使用ps 选项 | grep 'pattern'搜索

pidof可以根据程序名称查看进程pid

格式：

```bash
pgrep [options] pattern
```

常用选项：

- -u uid：生效者
- -U uid：进程发起者
- -p pid 显示指定进程的子进程

范例：

```bash
[root@centos8 ~]$pgrep -au user
447081 -bash
447115 bc
```

pidof查看pid

```bash
[root@centos8 ~]#pidof bash
19035 18813 18789 1251
[root@centos8 ~]#pidof ping.sh
[root@centos8 ~]#pidof -x ping.sh
19035
```

### 2.6 uptime

显示以下内容

- 当前系统时间
- 系统已经启动时间
- 系统在线用户数量
- 系统1分钟、5分钟和15分钟平均负载，建议超过5时告警

范例：

```bash
[root@centos8 ~]$uptime
 18:04:35 up 117 days, 18:50,  3 users,  load average: 0.00, 0.00, 0.00
[root@centos8 ~]$w
 18:04:45 up 117 days, 18:51,  3 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     pts/0    49.118.73.132    16:26    0.00s  0.10s  0.00s w
root     pts/1    49.118.73.132    16:28   12:05   0.02s  0.02s -bash
root     pts/2    49.118.73.132    17:05    4:37   0.03s  0.00s bc
```

### 2.7 top

查看实时的进程状态

h或？可以查看帮助，q或esc退出帮助

- 排序

  P：以占据的CPU百分比，%CPU

  M：以占据内存的百分比，%MEM

  T：累计占据CPU时长，TIME+

- 首部信息显示

  uptime信息：l

  tasks及cpu信息：t

  cpu分别显示：1

  memory信息：m

- 终止指定进程：k

- 保存文件：W

选项：

- -d # 指定刷新的时间间隔，默认为3秒
- -n # 刷新多少次以后退出

### 2.8 free

显示内存空间使用状态

格式：

```bash
free [OPTION]
```

常用选项：

- -b 以字节为单位
- -m 以MB为单位
- -g 以GB为单位

### 2.9 pmap

查看进程对应内存映射

格式：

```bash
pmap [options] pid [...]

cat /proc/PID/maps
```

范例：

```bash
#查看系统调用
[root@centos8 ~]$strace ls
execve("/usr/bin/ls", ["ls"], 0x7ffec1f150f0 /* 24 vars */) = 0
brk(NULL)                               = 0x55607ac10000
arch_prctl(0x3001 /* ARCH_??? */, 0x7ffd1a4a9aa0) = -1 EINVAL (Invalid argument)
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=31690, ...}) = 0
mmap(NULL, 31690, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f96cc1c5000
close(3)                                = 0
openat(AT_FDCWD, "/lib64/libselinux.so.1", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\240z\0\0\0\0\0\0"..., 832) = 832
lseek(3, 157536, SEEK_SET)              = 157536
read(3, "\4\0\0\0\20\0\0\0\5\0\0\0GNU\0\2\0\0\300\4\0\0\0\3\0\0\0\0\0\0\0", 32) = 32
fstat(3, {st_mode=S_IFREG|0755, st_size=304848, ...}) = 0
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f96cc1c3000
lseek(3, 157536, SEEK_SET)              = 157536
read(3, 
```

### 2.10 虚拟内存信息vmstat

格式：

```bash
vmstat [options] [delay [count]] 
```

显示说明：

```bash
procs:
 r：可运行（正运行或等待运行）进程的个数，和核心数有关
 b：处于不可中断睡眠态的进程个数(被阻塞的队列的长度)
memory：
 swpd: 交换内存的使用总量
 free：空闲物理内存总量
 buffer：用于buffer的内存总量
 cache：用于cache的内存总量
swap:
 si：从磁盘交换进内存的数据速率(kb/s)
 so：从内存交换至磁盘的数据速率(kb/s)
io：
 bi：从块设备读入数据到系统的速率(kb/s)
 bo: 保存数据至块设备的速率
system：
 in: interrupts 中断速率，包括时钟
 cs: context switch     进程切换速率
cpu：
 us:Time spent running non-kernel code
 sy: Time spent running kernel code
 id: Time spent idle. Linux 2.5.41前,包括IO-wait time.
 wa: Time spent waiting for IO. 2.5.41前，包括in idle.
 st: Time stolen from a virtual machine. 2.6.11前, unknown.
```

### 2.12 统计CPU和设备IO信息iostat

范例：

```bash
[root@centos8 ~]$iostat
Linux 4.18.0-193.14.2.el8_2.x86_64 (centos8.2) 	03/14/2021 	_x86_64_	(1 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.44    0.02    0.47    0.01    0.00   99.07

Device             tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               0.28         0.06         3.04     618723   31981155
```

### 2.13 系统资源统计 dstat

dstat由pcp-system-tools包提供，用于代替 vmstat,iostat功能

格式：

```bash
dstat [-afv] [options..] [delay [count]]
```

常见选项：

- -c 显示cpu相关信息
- -d 显示disk相关信息
- -m 显示memory相关统计数据

### 2.14 监视磁盘I/O iotop

格式：

```bash
iotop [OPTIONS]
```

常见选项：

- -d SEC, --delay=SEC设置每次监测的间隔，默认1秒，接受非整形数据例如1.1
- -u USER, --user=USER指定监测某个用户产生的I/O
- -P, --processes仅显示进程，默认iotop显示所有线程

交互按键

left和right方向键：改变排序

r：反向排序

o：切换至选项--only

p：切换至--processes选项

a：切换至--accumulated选项

q：退出

i：改变线程的优先级

### 2.15 显示网络带宽使用情况iftop

通过EPEL源安装

### 2.16 查看网络实时吞吐量nload

nload 是一个实时监控网络流量和带宽使用情况，以数值和动态图展示进出的流量情况,通过EPEL源安装

界面操作 上下方向键、左右方向键、enter键或者tab键都就可以切换查看多个网卡的流量情况

按 F2 显示选项窗口

按 q 或者 Ctrl+C 退出 nload

### 2.17 查看进程打开文件 lsof

查看当前系统文件的工具

格式：

```bash
lsof [options] [args]
```

常见选项：

- -a：列出打开文件存在的进程
- -c<进程名>：列出指定进程所打开的文件
- -g：列出GID号进程详情
- -d<文件号>：列出占用该文件号的进程
- +d<目录>：列出目录下被打开的文件
- -p<进程号>：列出指定进程号所打开的文件
- -u：列出UID号进程详情
- -i<条件>：列出符合条件的进程(4、6、协议、:端口、 @ip )

范例：

```bash
[root@centos8 ~]$lsof -i:22
COMMAND    PID USER   FD   TYPE  DEVICE SIZE/OFF NODE NAME
sshd       976 root    5u  IPv4   24095      0t0  TCP *:ssh (LISTEN)

[root@centos8 ~]$lsof -c bc
COMMAND    PID USER   FD   TYPE DEVICE SIZE/OFF      NODE NAME
bc      456068 user  cwd    DIR  253,1       83 100723329 /home/user
bc      456068 user  rtd    DIR  253,1      287       128 /
bc      456068 user  txt    REG  253,1    97256  34099041 /usr/bin/bc
bc      456068 user  mem    REG  253,1    36824  50333368 /usr/lib64/libdl-2.28.so
bc      456068 user  mem    REG  253,1  4176104  50333366 /usr/lib64/libc-2.28.so
bc      456068 user  mem    REG  253,1   208616  50333269 /usr/lib64/libtinfo.so.6.1
bc      456068 user  mem    REG  253,1   216912  50333259 /usr/lib64/libncurses.so.6.1
bc      456068 user  mem    REG  253,1   338648  50333641 /usr/lib64/libreadline.so.7.0
bc      456068 user  mem    REG  253,1   302552  50333359 /usr/lib64/ld-2.28.so
bc      456068 user  mem    REG  253,1   337024  16780403 /usr/lib/locale/en_US.utf8/LC_CTYPE
bc      456068 user  mem    REG  253,1    26998  16780842 /usr/lib64/gconv/gconv-modules.cache
bc      456068 user    0u   CHR  136,1      0t0         4 /dev/pts/1
bc      456068 user    1u   CHR  136,1      0t0         4 /dev/pts/1
bc      456068 user    2u   CHR  136,1      0t0         4 /dev/pts/1

#使用lsof恢复使用中误删除的文件
lsof |grep /var/log/messages
rm -f /var/log/messages
lsof |grep /var/log/messages
cat /proc/653/fd/6
cat /proc/653/fd/6 > /var/log/messages
```

### 2.18 信号发送 kill

格式：

```bash
kill [-signal|-s signal|-p] [-q value] [-a] [--] pid|name
```

常用选项：

- -SIGNAL
- -u uid: effective user，生效者
- -U uid: real user，真正发起运行命令者
- -t terminal: 与指定终端相关的进程
- -P pid: 显示指定进程的子进程

常用信号：

- 1）SIGHUP 无须关闭进程而让其重读配置文件
- 2）SIGINT 中止正在运行的进程；相当于Ctrl+c
- 3）SIGQUIT 相当于ctrl+\
- 9）SIGKILL 强制杀死正在运行的进程
- 1. SIGTERM 终止正在运行的进程，默认信号
- 1. SIGCONT 继续运行
- 1. SIGSTOP 后台休眠

指定信号的方法 :

- 信号的数字标识：1, 2, 9
- 信号的简写名称：HUP，hup
- 信号完整名称：SIGHUP，sighup

范例：

```bash
#利用 0 信号实现进程的健康性检查
[root@centos8 ~]#man kill
If signal is 0, then no actual signal is sent, but error checking is still
performed.

[root@centos8 ~]#killall -0 ping
[root@centos8 ~]#echo $?
0
#此方式有局限性，即使进程处于停止或僵尸状态，此方式仍然认为是进程是健康的
```

### 2.19 作业管理

Linux的作业控制

- 前台作业：通过终端启动，且启动后一直占据终端
- 后台作业：可通过终端启动，但启动后即转入后台运行（释放终端）

[![进程作业管理](https://www.cnblogs.com/bestvae/p/12.%E8%BF%9B%E7%A8%8B%E7%AE%A1%E7%90%86%E5%92%8C%E8%AE%A1%E5%88%92%E4%BB%BB%E5%8A%A1.assets/%E8%BF%9B%E7%A8%8B%E4%BD%9C%E4%B8%9A%E7%AE%A1%E7%90%86.png)](https://www.cnblogs.com/bestvae/p/12.进程管理和计划任务.assets/进程作业管理.png)

后台作业虽然被送往后台运行，但其依然与终端相关；退出终端，将关闭后台作业。如果希望送往后台 后，剥离与终端的关系可以采用如下方式：

- nohup COMMAND &>/dev/null &
- screen；COMMAND
- tmux；COMMAND

使用jobs命令可以查看`当前终端`所有作业

### 2.20 并行执行任务

利用后台执行，可以实现多个进程同时执行功能，从而提高执行效率，下面介绍3中方法。

方法一：

```bash
#在多个脚本后添加&符号实现并行执行多个脚本
[root@centos8 ~]$cat all.sh
f1.sh&
f2.sh&
f3.sh&
```

方法二：

```bash
(f1.sh&);(f2.sh&);(f3.sh&)
```

方法三：

```bash
f1.sh&f2.sh&f3.sh&
```

范例：

```bash
#利用&实现同时ping多个地址
[root@centos8 ~]$ping 127.1.1.1&ping 127.1.1.2
[1] 456646
PING 127.1.1.2 (127.1.1.2) 56(84) bytes of data.
64 bytes from 127.1.1.2: icmp_seq=1 ttl=64 time=0.020 ms
PING 127.1.1.1 (127.1.1.1) 56(84) bytes of data.
```

注意：由于使用&会使得进程进入后台运行，所以ctrl+c不能停止上例中进程，可以使用`killall ping`才可以结束

## 3 任务计划

任务计划就是让系统自动的按照时间或周期性执行任务，计划任务又分为一次性任务和周期性任务。

### 3.1 一次性任务

at：用于设定未来某个时间只执行一次的任务

格式：

```bash
at [option] TIME
Files:
		/var/spool/at				#at队列存放位置
      /etc/at.allow				#白名单：默认不存在，只有该文件中的用户才能执行at命令
      /etc/at.deny				#黑名单：y 默认存在，拒绝该文件中用户执行at命令
notes:		
		执行任务时PATH变量的值和当前定义任务的用户身份一致
		
		atd服务启动时才能实现at任务
		
		若/etc/at.deny,/etc/at.allow文件都不存在，只有 root 可以执行 at 命令allow优先级高于deny，当两个文		  件中都有同一个用户时，allow文件生效
```

常见选项：

- -l 列出指定队列中等待运行的作业；相当于atq
- -d 删除指定的作业；相当于atrm
- -c 查看具体作业任务
- -f /path/file 指定的文件中读取任务
- -m 当任务被完成之后，将给用户发送邮件，即使没有标准输出

时间格式：

```bash
HH:MM 在今日的 HH:MM 进行，若该时刻已过，则明天此时执行任务
02:00 

HH:MM YYYY-MM-DD   规定在某年某月的某一天的特殊时刻进行该项任务
02:00 2016-09-20 

HH:MM[am|pm] + number [minutes|hours|days|weeks]， 在某个时间点再加几个时间后才进行该
项任务
now + 5 min
02pm + 3 days
```

执行方式：

- 默认交互式执行
- 也可以配合管道符进行输入重定向
- -f 选项支持从文件读取

### 3.2 周期性任务计划 cron

cron任务分为

- 系统维护作业，/etc/crontab 主配置文件， /etc/cron.d/ 子配置文件
- 用户cron任务：保存在 /var/spool/cron/USERNAME，利用 crontab 命令管理

cron的日志位于：/var/log/cron

注意：cron 依赖于crond服务，确保crond守护处于运行状态

##### 3.2.1 系统cron计划任务

通过编辑/etc/crontab来制定系统计划任务

```bash
[root@centos8 ~]$cat /etc/crontab 
SHELL=/bin/bash								#shell类型
PATH=/sbin:/bin:/usr/sbin:/usr/bin		#若任务使用相对路径环境变量内的任务才能找到并被执行
MAILTO=root										#发送邮件用户
# For details see man 4 crontabs
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed

时间指定方式说明：
					* 			给定时间点上有效取值范围内的所有值
					#,#,#		离散时间值
					#-#		连续时间取值
					/#			在指定时间范围上，定义步长
example:
			#每晚21：10分使用user用户输出"Howdy!"
			10 21 * * * user /bin/echo "Howdy!"
			
			##每3小时echo和wall命令
			0 */3 * * * user /bin/echo “howdy”; wall “welcome!”
```

##### 3.2.2 用户计划任务

格式：

```bash
crontab [-u user] file
Files:
		/var/spool/cron/		#用户专用的cron任务文件
		/etc/cron.allow		#白名单作用于at类似
		/etc/cron.deny			#黑名单作用于at类似		
notes:
		crond是crontab的守护进程，执行crontab前请确保crond处于运行状态
		
		默认标准输出和错误会被发邮件给对应的用户,如：user创建的任务就发送至user的邮箱
		
		用户的cron 中默认 PATH=/usr/bin:/bin,如果使用其它路径,在任务文件的第一行加PATH=/path
```

常见选项：

- -l 列出所有任务
- -e 编辑任务
- -r 移除所有任务
- -i 同-r一同使用，以交互式模式移除指定任务
- -u user 仅root可运行，指定用户管理cron任务

注意：运行结果的标准输出和错误以邮件通知给相关用户

 cron任务中不建议使用%，它有特殊用途，它表示换行的特殊意义，且第一个%后的所有字符串会被将 成 当作命令的标准输入

范例：时间之间关系

```bash
[root@centos8 ~]#man 5 crontab
         Note: The day of a command's execution can be specified in the following two
         fields — 'day of   month', and 'day of week'. If both fields are restricted
         (i.e., do not contain the "*" character), the command will be run when either
         field matches the current time. 

example：
			30 4 1,15 * 5 user /bin/echo "Howdy!"
			#上面在每月1-15日和每周五都会执行
```

范例：让系统默认使用vim打开编辑任务文件

```bash
[root@centos8 ~]$ cat /etc/profile.d/env.sh
	export EDITOR=vim
```

思考：如果计划任务期间关机了，开机后任务还会执行么？

**anacron**

若计划任务期间系统处于关机状态，会 anacron来确保开机后自动执行未执行的计划任务由/etc/cron.hourly/0anacron执行，当执行任务时，更新/var/spool/anacron/cron.daily 文件的时间戳

配置文件：/etc/anacrontab 负责执行/etc/ cron.daily /etc/cron.weekly /etc/cron.monthly中系统任务

使用`man 5 anacrontab`查看/etc/anacrontab详细说明

##### 3.2.3 sleep

延迟命令执行时间，以秒为单位，可以实现在秒级别运行任务

格式：

```bash
sleep NUMBER[SUFFIX]
SUFFIX:
		 s: 秒, 默认
		 m: 分
		 h: 小时
		 d: 天
```

##### 3.2.4 管理临时文件

CentOS 7 使用 systemd-tmpfiles-setup服务实现

CentOS 6 使用/etc/cron.daily/tmpwatch定时清除临时文件

配置文件

```bash
/etc/tmpfiles.d/*.conf
/run/tmpfiles.d/*.conf
/usr/lib/tmpfiles/*.conf
```

##### 3.2.5 练习

1.显示统计占用系统内存最多的进程，并排序

```bash
[root@centos8 ~]$ps axo pid,cmd,%cpu,%mem,rss --sort -rss
    PID CMD                         %CPU %MEM   RSS
 334445 /usr/local/aegis/aegis_clie  0.5  3.1 58980
 556 /usr/lib/systemd/systemd-jo  0.0  2.4 44936
 808 /usr/libexec/sssd/sssd_nss   0.0  2.0 38296
 880 /usr/libexec/platform-pytho  0.0  1.5 28588
 718 /usr/lib/polkit-1/polkitd -  0.0  1.2 23116
```

2.编写脚本，使用for和while分别实现192.168.0.0/24网段内，地址是否能够ping通，若ping通则输出"sucess!",若ping不通则输出"fail!"

```bash
#for循环实现方式
[root@centos8 script]$cat for_ping.sh 
#!/bin/bash
for ip in {1..254};do
   {
    ping -c1 192.168.0.$ip > /dev/nvll && echo 192.168.0.$ip connect sucess! || echo 192.168.0.$ip connect fail!
    }&
done
   wait

#while循环实现方式
[root@centos8 script]$cat while_ping.sh 
#!/bin/bash
i=1
while [ $i -lt 255 ];do
    ( ping -c1 192.168.0.$i > /dev/null && echo 192.168.0.$i connect sucess! || echo 192.168.0.$i connect fail! )&
    i=$[i+1]
done
```

3.每周工作日1：30，将/etc备份至/backup目录中，保存文件名称格式为“etcbak-yyyy-mm-dd-HH.tar.xz”,其中日期是前一天的时间

```bash
[root@centos8 script]$cat backup.sh 
#!/bin/bash
tar -jcf /backup/"etcbak-`date -d '-1day' +%F-%H`.tar.xz" /etc/

[root@centos8 data]$crontab -l
30 1 * * 1-5 /data/script/backup.sh
```

4.工作日时间，每10分钟执行一次磁盘空间检查，一旦发现任何分区利用率高于80%，就发送邮件报警

```bash
[root@centos8 data]$crontab -l
/10 * * * 1-5 /data/script/check_disk
[root@centos8 ~]$cat /data/script/check_disk.sh 
#!/bin/bash
threshold=5
df -h | sed -rn '/^\/dev\//s#^([^ ]+).*([0-9]+)%.*#\1 \2#p' | while read Device Use;do
    [ $Use -gt $threshold ] && echo "$Device will be full,USE:$Use" | mail -s diskfull root
done
```

# 4.系统启动和服务管理

## 1 centos6的启动

### 1.1 Linux系统概述

- 内核功能：进程管理、内存管理、网络管理、驱动程序、文件系统、安全功能等
- 根文件系统（rootfs）：程序和 glibc 库

内核的两个流派：

- 宏内核(monolithic kernel)：又称单内核和强内核，Unix，Linux

  把所有系统服务都放到内核里，所有功能集成于同一个程序，分层实现不同功能，系统庞大复杂， Linux其实在单内核内核实现了模块化，也就相当于吸收了微内核的优点

- 微内核(micro kernel)：Windows，Solaris，HarmonyOS

  简化内核功能，在内核之外的用户态尽可能多地实现系统服务，同时加入相互之间的安全保护，每 种功能使用一个单独子系统实现，将内核功能移到用户空间，性能差

### 1.2 CentOS 6启动流程

##### 1.2.1 硬件启动

开机或重启主引导扇区的读取流程

1. BIOS加电自检（Power On Self Test -- POST）
2. 读取主引导记录（MBR）。当BIOS检查到硬件正常并与CMOS中的设置相符后，按照CMOS中对启动设备的设置顺序检测可用的启动设备。
3. 当检测到有启动设备满足要求后，BIOS将控制权交给相应启动设备。

加电自检数据保存位置：

- 主板的ROM，BIOS，这里保存着有关计算机系统最重要的基本输入输出 程序，系统信息设置、开机加电自检程序和系统启动自举程序等
- 主板的RAM，：CMOS互补金属氧化物半导体，这里保存各项参数的设定，按次序查找引导设备，第一个有引导程序的设备为本次启动设备

##### 1.2.2 启动加载器bootloader

BIOS寻找到第一个可启动的设备（通常为硬盘），而后从MBR中加载启动程序，然后把控制交给这段代码。MBR位于硬盘的前512字节内。

**引导程序**（英语：boot loader）是指引导操作系统的程序。第一阶段引导程序位于主引导记录（MBR），用以引导位于某个分区上的第二阶段引导程序，如NTLDR、BOOTMGR和GNU GRUB等。

主引导记录（MBR）最开头是第一阶段引导代码。其中的硬盘引导程序的主要作用是检查分区表是否正确并且在系统硬件完成自检以后将控制权交给硬盘上的引导程序

GRUB是Linux系统中所用的引导程序，这是一个来自GNU项目的多操作系统启动程序。

**GRUB 启动阶段**

- primary boot loader :

  1st stage：MBR的前446个字节

  1.5 stage： mbr 之后的扇区，让stage1中的bootloader能识别stage2所在的分区上的文件系统

- secondary boot loader ：2nd stage，分区文件/boot/grub/

GRUB的步骤1包含在MBR中。由于受MBR的大小限制，步骤一所做的几乎只是装载GRUB的下一步骤（存放在硬盘的其它位置）,也就是引出1.5或者2阶段。

##### 1.3.3 加载 kernel

kernel 自身初始化过程

1. 探测可识别到的所有硬件设备
2. 加载硬件驱动程序（借助于ramdisk加载驱动）
3. 以只读方式挂载根文件系统
4. 运行用户空间的第一个应用程序：/sbin/init

内核的组成部分：

- 核心文件：/boot/vmlinuz-VERSION-release

  ramdisk：辅助的伪根系统，加载相应的硬件驱动，ramdisk --> ramfs 提高速度

  CentOS 5 /boot/initrd-VERSION-release.img

  CentOS 6 以后版本 /boot/initramfs-VERSION-release.img

##### 1.3.4 init初始化

###### 1.3.4.1 运行级别

这个阶段首先通过init程序获取用户运行级别

不同版本系统init程序的类型：

CentOS 5之前

 配置文件：/etc/inittab

Upstart: init,CentOS 6

 配置文件：/etc/inittab, /etc/init/*.con

Systemd：systemd, CentOS 7

 配置文件：/usr/lib/systemd/system

 /etc/systemd/system

运行级别：为系统运行或维护等目的而设定；0-6：7个级别，一般使用3, 5做为默认级别

```bash
0：关机
1：单用户模式(root自动登录), single, 维护模式
2：多用户模式，启动网络功能，但不会启动NFS；维护模式
3：多用户模式，正常模式；文本界面
4：预留级别；可同3级别
5：多用户模式，正常模式；图形界面
6：重启
```

###### 1.3.4.2 初始化脚本 sysinit

```bash
/etc/rc.d/rc.sysinit
```

初始化脚本的功能：

```bash
[root@centos6 ~]#cat /etc/init/rcS.conf 
(1) 设置主机名
(2) 设置欢迎信息
(3) 激活udev和selinux
(4) 挂载/etc/fstab文件中定义的文件系统
(5) 检测根文件系统，并以读写方式重新挂载根文件系统
(6) 设置系统时钟
(7) 激活swap设备
(8) 根据/etc/sysctl.conf文件设置内核参数
(9) 激活lvm及software raid设备
(10)加载额外设备的驱动程序
(11)清理操作
```

###### 1.3.4.3 启动服务

执行运行级别下服务的启动脚本

```bash
服务脚本放置于/etc/rc.d/init.d (/etc/init.d)
/etc/rc.d/rc.local
```

service 命令：手动管理服务

```bash
service 服务 start|stop|restart
service --status-all
```

chkconfig命令配置服务开机启动：

```bash
#查看服务在所有级别的启动或关闭设定情形：
chkconfig [--list] [name]

#添加服务
SysV的服务脚本放置于/etc/rc.d/init.d (/etc/init.d)
#!/bin/bash
chkconfig: LLLL nn nn  #LLLL 表示初始在哪个级别下启动，-表示都不启动
description : 描述信息

chkconfig --add name

#删除服务
chkconfig --del name

#修改指定的运行级别
chkconfig [--level levels] name <on|off|reset>
说明：--level LLLL: 指定要设置的级别；省略时表示2345
```

###### 1.3.4.4 非独立服务管理

服务分为独立服务和非独立服务

瞬态（Transient）服务被超级守护进程 xinetd 进程所管理，也称为非独立服务，这类服务在被使用时才会被唤醒。

配置文件：

```bash
/etc/xinetd.conf
/etc/xinetd.d/<service>
```

用chkconfig控制非独立服务开机启动

示例：chkconfig tftp on

## 2 /proc 目录和内核参数管理

/proc目录：内核把自己内部状态信息及统计信息，以及可配置参数通过proc伪文件系统加以输出

查看/proc帮助：man proc

/proc设置的方法：

```bash
#使用命令设置，下次开机后失效
sysctl -w path.to.parameter=VALUE

#写入配置文件，持久保存配置，下面任意一个配置文件即可
/etc/sysctl.conf 
/etc/sysctl.d/*.conf
/usr/local/lib/sysctl.d/*.conf
/usr/lib/sysctl.d/*.conf
/lib/sysctl.d/*.conf
/run/sysctl.d/*.conf

#echo命令通过重定向方式也可以修改大多数参数的值
echo "VALUE" > /proc/sys/path/to/parameter
```

sysctl命令用法：

临时设置参数

```bash
sysctl -w parameter=VALUE
```

通过读取配置文件设置参数

```bash
sysctl -p [/path/to/conf_file]
```

查看所有生效参数

```bash
sysctl -a
```

常用的内核参数：

```
net.ipv4.ip_forward			#路由转发功能
net.ipv4.icmp_echo_ignore_all
net.ipv4.ip_nonlocal_bind   #允许应用程序可以监听本地不存在的IP
vm.drop_caches
fs.file-max = 1020000		#允许打开的最大文件数
```

/sys 目录

使用sysfs文件系统，为用户使用的伪文件系统，输出内核识别出的各硬件设备的相关属性信息，也有内 核对硬件特性的设定信息；有些参数是可以修改的，用于调整硬件工作特性

udev通过此路径下输出的信息动态为各设备创建所需要设备文件

udev为设备创建设备文件时，会读取其事先定义好的规则文件，一般在/etc/udev/rules.d 及/usr/lib/udev/rules.d目录下

## 3 内核模块管理

Linux是单内核体系设计、但充分借鉴了微内核设计体系的优点，为内核引入模块化机制 内核组成部分：

kernel：内核核心，一般为bzImage，通常在/boot目录下

 名称为 vmlinuz-VERSION-RELEASE

kernel object：内核对象，一般放置于

 /lib/modules/VERSION-RELEASE/

使用`uname -r`可以查看内核版本

### 3.1 内核模块命令

##### **lsmod**

- 显示由核心已经装载的内核模块
- 显示的内容来自于: /proc/modules文件

##### **modinfo**

查看内核模块

格式：

```bash
modinfo [ -k kernel ] [ modulename|filename... ]
```

常用选项：

- -n：只显示模块文件路径
- -p：显示模块参数
- -a：作者
- -d：描述

##### modprobe

添加或者移除内核模块

格式：

```bash
modprobe [-C config-file] [modulename] [module parameters...]
modprobe [-r] [-v] [-n] [-i] [modulename...]
```

常用选项：

- -r 移除指定模块

范例：

```bash
[root@centos8 ~]$modprobe uas
[root@centos8 ~]$lsmod | grep usb
usb_storage            73728  1 uas
[root@centos8 ~]$modprobe -r usb_storage 
modprobe: FATAL: Module usb_storage is in use.
[root@centos8 ~]$modprobe -r uas
[root@centos8 ~]$lsmod | grep usb
[root@centos8 ~]$
```

其他命令：

depmod命令：内核模块依赖关系文件及系统信息映射文件的生成工具

insmod命令：可以安装模块，需要指定模块文件路径，并且不自动解决依赖模块

范例：

```bash
insmod `modinfo –n exportfs`
insmod `modinfo –n xfs`
```

rmmod命令：卸载模块

```bash
rmmod [ modulename ]
```

## 4 Busybox

### 4.1 Busybox介绍

Busybox 是一个集成了三百多个最常用Linux命令和工具的软件。同时它自身只占用很小的空间

可以用于定制小型的Linux操作系统：linux内核+busybox

官方网站：https://busybox.net/

### 4.2 busybox编译安装

```bash
[root@centos7 ~]#yum -y install   gcc gcc-c++ glibc glibc-devel make pcre pcredevel openssl openssl-devel systemd-devel zlib-devel glibc-static ncurses-devel

[root@centos7 ~]#tar xvf busybox-1.31.1.tar.bz2
[root@centos7 ~]#cd busybox-1.31.1/
[root@centos7 busybox-1.31.1]#make menuconfig 

#按下面选择，把busybox编译也静态二进制、不用共享库:Settings -->Build Options -->[*] Build static binary (no sharedlibs)
[root@centos7 busybox-1.31.1]#make 

#如果出错，执行make clean后，重新执行上面命令
[root@centos7 busybox-1.31.1]#make install 
[root@centos7 busybox-1.31.1]#ll busybox -h
-rwxr-xr-x 1 root root 2.6M May 13 14:46 busybox

#在install目录中创建了各种命令的软链接便于使用
[root@centos7 busybox-1.31.1]#ls _install/
bin linuxrc sbin usr
```

### 4.3 Busybox使用

busybox的使用有三种方式：

- busybox后直接跟命令，如 busybox ls
- 直接将busybox重命名，如 cp busybox tar
- 创建符号链接，如 ln -s busybox rm

## 5 systemd

### 5.1 systemd介绍

Systemd：从 CentOS 7 版本之后开始用 systemd 实现init进程，系统启动和服务器守护进程管理器， 负责在系统启动或运行时，激活系统资源，服务器进程和其它进程

**Systemd新特性**

- 按需启动守护进程
- 自动化的服务依赖关系管理
- 并行执行系统服务等

systemd概念之：unit

unit表示不同类型的systemd对象，通过配置文件进行标识和配置；文件中主要包含了系统服务、监听 socket、保存的系统快照以及其它与init相关的信息

Unit常用类型：

- service ：文件扩展名为.service, 用于定义系统服务
- socket ：定义进程间通信用的socket文件，也可在系统启动时，延迟启动服务，实现 按需启动
- target：文件扩展名为.target，用于模拟实现运行级别

使用`systemctl -t help`可以查看unit全部类型

unit的配置文件

```bash
/usr/lib/systemd/system:每个服务最主要的启动脚本设置，类似于之前的/etc/init.d/
/lib/systemd/system: ubutun的对应目录

/run/systemd/system：系统执行过程中所产生的服务脚本，比上面目录优先运行
/etc/systemd/system：管理员建立的执行脚本，类似于/etc/rcN.d/Sxx的功能，比上面目录优先运行
```

更为详细信息可以访问红帽官网查看：

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/chap-managing_services_with_systemd

http://freedesktop.org/wiki/Software/systemd/Preset

### 5.2 systemctl管理系统服务

格式：

```bash
systemctl COMMAND name.service
```

常用命令列举：

```bash
#启动：相当于service name start
systemctl start name.service  

#停止：相当于service name stop
systemctl stop name.service

#重启：相当于service name restart
systemctl restart name.service

#查看状态：相当于service name status
systemctl status name.service

#禁止自动和手动启动：
systemctl mask name.service
这条命令本质上是将/etc/systemd/system/name.service软链接至/dev/null

#取消禁止
systemctl unmask name.service

#查看某服务当前激活与否的状态：
systemctl is-active name.service

#查看所有已经激活的服务：
systemctl list-units --type|-t service

#查看所有服务：
systemctl list-units --type service --all|-a

#设定某服务开机自启，相当于chkconfig name on
systemctl enable name.service 

#查看所有服务的开机自启状态，相当于chkconfig --list
systemctl list-unit-files --type service

#用来列出该服务在哪些运行级别下启用和禁用：chkconfig –list name
ls /etc/systemd/system/*.wants/name.service

#查看服务是否开机自启：
systemctl is-enabled name.service
```

服务状态说明：

- active(running) 一次或多次持续处理的运行
- inactive 不运行
- static 开机不启动，但可被另一个启用的服务激活
- enabled 开机启动
- disabled 开机不启动

### 5.3 service unit文件格式

/etc/systemd/system：系统管理员和用户使用 （其中内容是/usr/lib/systemd/system软链接）

/usr/lib/systemd/system：发行版打包者使用

unit 格式说明：

- 以 “#” 开头的行后面的内容会被认为是注释
- 相关布尔值，1、yes、on、true 都是开启，0、no、off、false 都是关闭
- 时间单位默认是秒，所以要用毫秒（ms）分钟（m）等须显式说明

service unit file文件通常由三部分组成

```bash
[root@Centos7 ~]# cat  /usr/lib/systemd/system/NetworkManager.service 
#有关unit的信息相关选项
[Unit]
...

#unit类型选项集合
[Service]
...

#定义由“systemctl enable”以及"systemctl disable“命令在实现服务启用或禁用时用到的一些选项
[Install]
...
```

[Unit]的常用选项：

- Description：描述信息
- After：定义unit的启动次序，表示当前unit应该晚于哪些unit启动，其功能与Before相反
- Requires：依赖到的其它units，强依赖，被依赖的units无法激活时，当前unit也无法激活
- Wants：依赖到的其它units，弱依赖
- Conflicts：定义units间的冲突关系

[Service]的常用选项：

- Type：定义影响ExecStart及相关参数的功能的unit进程启动类型

  simple：默认值，这个daemon主要由ExecStart接的指令串来启动，启动后常驻于内存中

- EnvironmentFile：环境配置文件

- ExecStart：指明启动unit要运行命令或脚本的绝对路径

- ExecStartPre： ExecStart前运行

- ExecStartPost： ExecStart后运行

- ExecStop：指明停止unit要运行的命令或脚本

- Restart：当设定Restart=1 时，则当次daemon服务意外终止后，会再次自动启动此服务

- PrivateTmp：设定为yes时，会在生成/tmp/systemd-private-UUID-NAME.service-XXXXX/tmp/ 目录

[Install]的常用选项:

- Alias：别名，可使用systemctl command Alias.service
- RequiredBy：被哪些units所依赖，强依赖
- WantedBy：被哪些units所依赖，弱依赖

范例：开机启动nginx

```bash
#方式1：
[root@Centos7 ~]# vim   /usr/lib/systemd/system/tomcat.service
[Unit]
Description=java tomcat project
After=syslog.target network.target

[Service]
Type=forking
EnvironmentFile=/usr/local/tomcat/conf/tomcat.conf
ExecStart=/usr/local/tomcat/bin/startup.sh
ExecStop=/usr/local/tomcat/bin/shutdown.sh
PrivateTmp=true
User=tomcat

[Install]
WantedBy=multi-user.target

#方式2：
[root@Centos7 ~]#vim /etc/rc.local
[root@Centos7 ~]#cat /etc/rc.local
#!/bin/bash
/usr/local/tomcat/bin/startup.sh
```

### 5.4 运行级别

target units：相当于CentOS 6之前的runlevel ,unit配置文件：.target

```bash
ls /usr/lib/systemd/system/*.target
systemctl list-unit-files --type target --all
```

级别对应关系

```bash
0 ==> runlevel0.target, poweroff.target
1 ==> runlevel1.target, rescue.target
2 ==> runlevel2.target, multi-user.target
3 ==> runlevel3.target, multi-user.target
4 ==> runlevel4.target, multi-user.target
5 ==> runlevel5.target, graphical.target
6 ==> runlevel6.target, reboot.target
```

常用命令：

```bash
#级别切换：相当于 init N
systemctl isolate name.target

#获取默认运行级别： 相当于查看 /etc/inittab
systemctl get-default

#修改默认级别：相当于修改 /etc/inittab
systemctl set-default name.target
```

### 5.5 CentOS 7之后版本引导顺序

大图：http://www.bestvae.cn/centos7.svg

[![centos7启动流程](https://www.cnblogs.com/bestvae/p/13.%E7%B3%BB%E7%BB%9F%E5%90%AF%E5%8A%A8%E5%92%8C%E6%9C%8D%E5%8A%A1%E7%AE%A1%E7%90%86.assets/centos7%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B.png)](https://www.cnblogs.com/bestvae/p/13.系统启动和服务管理.assets/centos7启动流程.png)

### 5.6 设置内核参数

设置内核参数，只影响当次启动

常用于实现密码的破解

启动时，到启动菜单，按e键，找到在linux 开头的行后添加systemd.unit=desired.target

### 5.7 练习

##### 5.7.1 自制一个只运行shell的linux系统

**参考的文档：**

Grub配置

https://www.gnu.org/software/grub/manual/grub/grub.html#Invoking-grub_002dmkrelpath

http://www.jinbuguo.com/linux/grub.cfg.html

内核相关参数设置

https://www.kernel.org/doc/html/v4.14/admin-guide/kernel-parameters.html

1、硬件环境准备

```bash
#此次实验系统版本为CentOS Linux release 7.9.2009 (Core)

#1)创建boot和/分区
[root@localhost ~]# fdisk /dev/sdb
#创建boot分区
Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-41943039, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-41943039, default 41943039): +1G
Partition 1 of type Linux and of size 1 GiB is set
#创建/分区
Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p
Partition number (2-4, default 2): 2
First sector (2099200-41943039, default 2099200): 
Using default value 2099200
Last sector, +sectors or +size{K,M,G} (2099200-41943039, default 41943039): 
Using default value 41943039
Partition 2 of type Linux and of size 19 GiB is set
#保存设置退出
Command (m for help): w

#2)格式文件系统
[root@localhost ~]# mkfs.ext4 /dev/sdb1
[root@localhost ~]# mkfs.ext4 /dev/sdb2

#3)挂载磁盘设备
[root@localhost ~]# mount /dev/sdb1 /mnt/boot
[root@localhost ~]# mount /dev/sdb2 /mnt/sysroot/
```

2、配置grub

```bash
#1)安装grub
[root@localhost ~]# grub2-install --root-directory=/mnt/ /dev/sdb
Installing for i386-pc platform.
Installation finished. No error reported.
[root@localhost ~]# ls /mnt/boot/
grub2  lost+found
#也可以使用下面这个命令安装grub
[root@localhost ~]#grub2-install --boot-directory=/mnt/boot /dev/sdb --force

#2)拷贝内核以及虚拟文件系统
[root@localhost ~]# cp /boot/vmlinuz-3.10.0-1160.el7.x86_64 /mnt/boot/
[root@localhost ~]# cp /boot/initramfs-3.10.0-1160.el7.x86_64.img /mnt/boot/

#3)自定义名称、在选择菜单界面等待时间6s，指定init程序为bash
[root@localhost /]# cat /etc/default/grub 
#通常来说建议在这个文件设置grub相关参数，再运行grub2-mkconfig生成grub.cfg文件
GRUB_TIMEOUT=6		#选择菜单界面等待秒数
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="rhgb quiet root=/dev/sda2 selinux=0 init=/bin/bash"		#向内核添加参数
GRUB_DISABLE_RECOVERY="true"
[root@localhost /]# grub2-mkconfig -o /mnt/boot/grub2/grub.cfg
[root@localhost /]# cat /mnt/boot/grub2/grub.cfg
menuentry 'CentOS Linux this is for test' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-1160.el7.x86_64-advanced-b3124ba9-2b20-4b11-915d-df97a9e8a256' {
        load_video
        set gfxpayload=keep
        insmod gzio
        insmod part_msdos
        insmod xfs
        set root='hd0,msdos1'			#指定根设备，任何未指定设备名的文件都视为位于此设备
        if [ x$feature_platform_search_hint = xy ]; then
          search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos1 --hint-efi=hd0,msdos1 --hint-baremetal=ahci0,msdos1 --hint='hd0,msdos1' 4d3af8d0-7296-488d-96b6-725d5c5d9152 
        else
          search --no-floppy --fs-uuid --set=root 4d3af8d0-7296-488d-96b6-725d5c5d9152
        fi
        linux16 /vmlinuz-3.10.0-1160.el7.x86_64 root=/dev/sda2 ro rhgb quiet selinux=0 init=/bin/bash 	
        initrd16 /initramfs-3.10.0-1160.el7.x86_64.img
}
```

3、准备相关程序和库

```bash
[root@localhost /]#mkdir /mnt/sysroot
[root@localhost /]#mount /dev/sdb2   /mnt/sysroot
[root@localhost /]# mkdir -pv /mnt/sysroot/{boot,dev,sys,proc,etc,lib,lib64,bin,sbin,tmp,var,usr,opt,home,root,mnt,media}
#使用脚本复制bash等命令和相关库文件，如：bash,ifconfig,insmod,ping,mount,ls,cat,df,lsblk,blkid,tree,fdisk
[root@localhost data]# cat copycmd.sh 
#!/bin/bash
ch_root="/mnt/sysroot"
[ ! -d $ch_root ] && mkdir $ch_root
 
bincopy() {
    if which $1 &>/dev/null; then

        local cmd_path=`which --skip-alias $1`
        local bin_dir=`dirname $cmd_path`
        [ -d ${ch_root}${bin_dir} ] || mkdir -p ${ch_root}${bin_dir}
        [ -f ${ch_root}${cmd_path} ] || cp $cmd_path ${ch_root}${bin_dir}
        return 0
    else
        echo "Command not found."
        return 1
    fi
}
 
libcopy() {
    local lib_list=$(ldd `which --skip-alias $1` | grep -Eo '/[^[:space:]]+')
    for loop in $lib_list;do
        local lib_dir=`dirname $loop`
        [ -d ${ch_root}${lib_dir} ] || mkdir -p  ${ch_root}${lib_dir}
        [ -f ${ch_root}${loop} ] || cp $loop ${ch_root}${lib_dir}
    done
} 
read -p "Please input a command: " command
 
while [ "$command" != "quit" ];do
    if bincopy $command ;then
        libcopy $command
    fi
    read -p "Please input a command or quit: " command
done
[root@localhost data]#chroot /mnt/sysroot
bash-4.2# ls
bash-4.2# pwd 
/
```

到这里配置就完成了，现在只需新建一个虚拟机，将前一虚拟机sdb硬盘对应的vmdk文件增加进去开机即可

[![自制Linux](https://www.cnblogs.com/bestvae/p/13.%E7%B3%BB%E7%BB%9F%E5%90%AF%E5%8A%A8%E5%92%8C%E6%9C%8D%E5%8A%A1%E7%AE%A1%E7%90%86.assets/%E8%87%AA%E5%88%B6Linux.jpg)](https://www.cnblogs.com/bestvae/p/13.系统启动和服务管理.assets/自制Linux.jpg)

##### 5.7.2 破解centos7密码

方法一：

```bash
启动时任意键暂停启动
按e键进入编辑模式
将光标移动linux 开始的行，添加内核参数rd.break
按ctrl-x启动
mount –o remount,rw /sysroot
chroot /sysroot
passwd root
#如果SELinux是启用的,才需要执行下面操作,如查没有启动,不需要执行
touch /.autorelabel
exit
reboot
```

方法二：

```bash
启动时任意键暂停启动
按e键进入编辑模式
将光标移动linux 开始的行，改为rw init=/sysroot/bin/sh
按ctrl-x启动
chroot /sysroot
passwd root
#如果SELinux是启用的,才需要执行下面操作,如查没有启动,不需要执行
touch /.autorelabel
exit
reboot
```

# 5.加密技术和安全

## 1 加密技术

### 1.1 对称加密算法

对称加密：加密和解密使用同一个密钥

特性：

- 加密、解密使用同一个密钥，效率高
- 将原始数据分割成固定大小的块，逐个进行加密

缺点：

- 不同对象间会产生不同密钥，造成密钥数过多
- 密钥不好分发给对方
- 无法确认数据来源

常见的对称加密算法：

- 3DES：
- AES：Advanced (128, 192, 256bits)

### 1.2 非对称加密算法

非对称加密：密钥是成对出现，分为公钥和私钥

- 公钥：public key，公开给所有人，主要给别人加密使用
- 私钥：secret key，private key 自己留存，必须保证其私密性，用于自已加密签名

注意：用公钥加密数据，只能使用与之配对的私钥解密

主要功能：

- 数字签名：接收方能够确认发送发身份
- 数字加密：可用于加密对称密钥

缺点：

- 密钥较长，算法较复杂
- 加密和解密效率低

常见算法：

- RSA：由 RSA 公司发明，是一个支持变长密钥的公共密钥算法，需要加密的文件块的长度也是可 变的,可实现加密和数字签名
- DSA（Digital Signature Algorithm）：数字签名算法，是一种标准的 DSS（数字签名标准）
- ECC（Elliptic Curves Cryptography）：椭圆曲线密码编码学，比RSA加密算法使用更小的密钥， 提供相当的或更高等级的安全

### 1.3 单向哈希算法

哈希算法：也称为散列算法，将任意数据缩小成固定大小的“指纹”，称为digest，即摘要

特点：

- 任意长度输入，固定长度输出
- 若修改数据，指纹也会改变，且有雪崩效应，数据的一点微小改变，生成的指纹值变化非常大。
- 无法从指纹中重新生成数据，即不要逆，具有单向性

功能：用于校验数据完整性

常见算法

md5: 128bits、sha1: 160bits、sha224 、sha256、sha384、sha512

常用工具

- md5sum | sha1sum [ --check ] file
- openssl、gpg
- rpm -V

### 1.5 多种加密方式

#### 1.5.1 实现数据加密

[![实现数据加密](https://img2020.cnblogs.com/blog/2222624/202103/2222624-20210330221250557-846529333.jpg)](https://img2020.cnblogs.com/blog/2222624/202103/2222624-20210330221250557-846529333.jpg)

#### 1.5.2 实现数字签名

[![实现数据加密](https://img2020.cnblogs.com/blog/2222624/202103/2222624-20210330221302009-751818679.jpg)](https://img2020.cnblogs.com/blog/2222624/202103/2222624-20210330221302009-751818679.jpg)

#### 1.5.3 综合加密和签名

**方式一：数字签名和非对称加密**组合

A（Alice）：hash原文→A私钥加密摘要成为数字签名→B公钥加密原文和数字签名→传送给Bob

B（Bob）：B私钥解密成数字签名和原文→A公钥解密数字签名称为摘要和原文→对比A和B的原文摘要

**方式二：数字签名和对称加密组合**

A（Alice）：hash原文→A私钥加密摘要成为数字签名→对称加密原文和数字签名→公钥B加密对称密钥→传送给Bob

B（Bob）：B私钥解密对称密钥→对称密钥解密→A公钥解密数字签名→对比A和B的原文摘要

### 1.6 CA和证书

#### 1.6.1 CA和证书

CA和证书主要是为了防止中间人将服务端返回的公钥替换为自己公钥，其工作原理如下：

[![实现数据加密](https://myblognote.oss-cn-beijing.aliyuncs.com/linux%E5%9F%BA%E7%A1%80%E5%8F%8A%E7%B3%BB%E7%BB%9F%E5%AE%89%E8%A3%85/CA%E5%92%8C%E8%AF%81%E4%B9%A6.jpg)](https://myblognote.oss-cn-beijing.aliyuncs.com/linux基础及系统安装/CA和证书.jpg) PKI：Public Key Infrastructure 公共密钥加密体系

签证机构：CA（Certificate Authority）

注册机构：RA

证书吊销列表：CRL

#### 1.6.2 HTTPS

HTTPS 协议：就是“HTTP 协议”和“SSL/TLS 协议”的组合。

[![实现数据加密](https://img2020.cnblogs.com/blog/2222624/202103/2222624-20210330221034862-98307978.jpg)](https://img2020.cnblogs.com/blog/2222624/202103/2222624-20210330221034862-98307978.jpg)

从上图我们知道HTTPS无非是在传输层和应用层中间加了一层TLS

#### 1.6.3 安全协议 SSL/TLS

1、SSL/TLS：是为网络通信提供安全及数据完整性的一种传输层安全协议

- SSL/TLS协议的基本思路是采用公钥加密法，也就是说，客户端先向服务器端索要公钥，然后用公钥加密信息，服务器收到密文后，用自己的私钥解密。

功能：

- 机密性：SSL协议使用密钥加密通信数据。
- 可靠性：服务器和客户都会被认证，客户的认证是可选的。
- 完整性：SSL协议会对传送的数据进行完整性检查。

2、TLS组成：

- Handshake协议：包括协商安全参数和密码套件、服务器身份认证（客户端身份认证可选）、密 钥交换
- ChangeCipherSpec 协议：一条消息表明握手协议已经完成
- Alert 协议：对握手协议中一些异常的错误提醒，分为fatal和warning两个级别，fatal类型错误会 直接中断SSL链接，而warning级别的错误SSL链接仍可继续，只是会给出错误警告
- Record 协议：包括对消息的分段、压缩、消息认证和完整性保护、加密等

3、TLS基本的运行过程（HTTPS工作过程）

实现分为握手阶段和应用阶段

- 握手阶段(协商阶段):客户端和服务器端认证对方身份（依赖于PKI体系，利用数字证书进行身份认 证），并协商通信中使用的安全参数、密码套件以及主密钥。后续通信使用的所有密钥都是通过 MasterSecret生成
- 应用阶段:在握手阶段完成后进入，在应用阶段通信双方使用握手阶段协商好的密钥进行安全通信

握手阶段详细过程：

这个阶段总共涉及四次通信，而且此阶段所有通信都是明文

1. 客户端发出请求**（ClientHello）**

   客户端首先向服务器发出加密通信请求，称为ClientHello。

   这一步客户端提供

   - TLS协议版本
   - 一个客户端生成的随机数
   - 支持加密的方法
   - 支持压缩的方法

2. 服务端回应（ServerHello）

   服务端向客户端发出回应，回应包含

   - 确认使用的加密通信协议版本
   - 服务器生成的随机数
   - 确认加密方法
   - 服务器证书

3. 客户端回应

   客户端收到服务器回应后，会先验证服务器证书是否可信，不可信则会告警，接着会从证书中取出服务器公钥，再向服务器发送三项信息

   - 一个随机数，这个随机数会用服务器公钥加密
   - 编码改变通知，表示以后双方发送的信息会用协商好的加密方式发送
   - 客户端握手结束通知，这项也是前面发送所有内容的hash值，用来供服务器校验。

   整个阶段出现了三个随机数，主要是为了使得加密更安全，不容易被破解

4. 服务器回应

   服务器收到客户端第三个随机数后，利用者三个数计算生成所用的“会话密钥”，向客户端发送信息

   - 编码改变通知
   - 服务器握手结束通知，这项也是前面发送所有内容的hash值，用来供客户端校验。

至此握手阶段全部结束，接下来使用会话密钥进入“加密通信”，不过使用的是普通HTTP协议

[![img](https://img2020.cnblogs.com/blog/2222624/202103/2222624-20210330221158579-568262950.png)](https://img2020.cnblogs.com/blog/2222624/202103/2222624-20210330221158579-568262950.png)

目前密钥交换 + 签名有三种主流选择：

- RSA 密钥交换、RSA 数字签名
- ECDHE 密钥交换、RSA 数字签名
- ECDHE 密钥交换、ECDSA 数字签名

上面所描述握手采用第一种方式，整个握手阶段都不加密（也没法加密），都是明文的。因此，如果有人窃听通信，他可以知道双方选择的加密方法，以及三个随机数中的两个。整个通话的安全，只取决于第三个随机数（Premaster secret）能不能被破解。

虽然理论上，只要服务器的公钥足够长（比如2048位），那么Premaster secret可以保证不被破解。但是为了足够安全，我们可以考虑把握手阶段的算法从默认的RSA算法，改为 Diffie-Hellman算法（简称DH算法）。

采用DH算法后，Premaster secret不需要传递，双方只要交换各自的参数，就可以算出这个随机数。

[![img](https://img2020.cnblogs.com/blog/2222624/202103/2222624-20210330221216783-1669820648.png)](https://img2020.cnblogs.com/blog/2222624/202103/2222624-20210330221216783-1669820648.png)

上图中，第三步和第四步由传递Premaster secret变成了传递DH算法所需的参数，然后双方各自算出Premaster secret。

参考链接：

http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html

http://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html

https://tools.ietf.org/html/rfc5246

## 2 openssl

### 2.1 openssl介绍

**OpenSSL**是一个开放源代码的软件库包，应用程序可以使用这个包来进行安全通信，避免窃听，同时确认另一端连线者的身份。

其主要库是以C语言所写，实现了基本加密功能，实现了SSL与TLS协议

官方站点：[https://www.openssl.org](https://www.openssl.org/)

openssl官方推荐电子书：https://www.feistyduck.com/library/openssl-cookbook/

### 2.2 Base64编码

**Base64**是一种基于64个可打印字符来表示二进制数据的表示方法。每6位二进制数为一个单元。3个字节共有24个二进制位，所以对应64个Base64单元，也就是说3个字节可由4个可打印字符表示。

如果要编码的字节数不能被3整除，最后会多出1个或2个字节，那么可以使用下面的方法进行处理：先使用0字节值在末尾补足，使其能够被3整除，然后再进行Base64的编码。

参考：https://zh.wikipedia.org/wiki/Base64

范例：

```bash
[root@centos8 ~]#echo -n Man | base64
TWFu
[root@centos8 ~]#echo TWFu | base64 -d
Man[root@centos8 ~]#
[root@centos8 ~]#echo -n ab | base64
YWI=
[root@centos8 ~]#echo -n ab | base64 | base64 -d
ab[root@centos8 ~]#
```

### 2.3 openssl命令

两种运行模式：

- 交互模式
- 批处理模式

三种子命令：

- 标准命令
- 消息摘要命令
- 加密命令

范例：

```bash
[root@centos8 ~]# openssl version
OpenSSL 1.1.1c FIPS  28 May 2019
[root@centos8 ~]# openssl help
Standard commands
asn1parse         ca                ciphers           cms               
crl               crl2pkcs7         dgst              dhparam           
dsa               dsaparam          ec                ecparam           
enc               engine            errstr            gendsa            
......          

Message Digest commands (see the `dgst' command for more details)        
sha3-384          sha3-512          sha384            sha512            
sha512-224        sha512-256        shake128          shake256          
sm3               
......

Cipher commands (see the `enc' command for more details)
aes-128-cbc       aes-128-ecb       aes-192-cbc       aes-192-ecb       
......           
            

[root@centos8 ~]# openssl
OpenSSL> version
OpenSSL 1.1.1c FIPS  28 May 2019
```

#### 2.3.1 openssl命令对称加密

命令：

openssl enc

算法：

3des, aes, blowfish, twofish

帮助：man enc

选项：

- -a 使用base64对加密后解密前的数据进行处理
- -e 加密输入数据。默认选项
- -des3 DES（Data Encryption Standard），用DES3算法的CBC模式加密文件
- -salt 加密时加入随机生成数
- -in,-out 输入输出文件

范例：

```bash
#加密
[root@centos8 data]# openssl enc -des3 -a -salt -in test.txt -out test1
enter des-ede3-cbc encryption password:
Verifying - enter des-ede3-cbc encryption password:
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
[root@centos8 data]# cat test1
U2FsdGVkX18/V5eX2Mqt8dzbLtLUyx3H

#解密
[root@centos8 data]# openssl enc -d -des3 -a -in test1 -out test2
enter des-ede3-cbc decryption password:
*** WARNING : deprecated key derivation used.
Using -iter or -pbkdf2 would be better.
[root@centos8 data]# cat test2 
123
```

#### 2.3.2 openssl命令单向哈希加密

命令：

openssl dgst

算法：

md5sum, sha1sum, sha224sum,sha256sum…

帮助：man dgst

范例：

```bash
#用MD5算法计算文件fstab的哈西值，输出到stdout：
[root@centos8 data]# openssl dgst -md5 fstab 
MD5(fstab)= ab3ee666e086e8dac26f96079ac5e63d
```

#### 2.3.3 openssl命令生成用户密码

帮助：man sslpasswd

范例：

```bash
#以交互方式生成
[root@centos8 data]# openssl passwd -6 -salt qweewq -stdin
xiaoming
$6$qweewq$Uy5hJssuD8tzMmnK2S.7rYLgveMlQ0uvOwsJR2.oyNCjd7ksEt2K6SE7VK5toP1jnyQzQRj0HLapvScAECSKE/

#以非交互式生成
[root@centos8 data]# echo xiaoming | openssl passwd -6 -salt qweewq -stdin
$6$qweewq$Uy5hJssuD8tzMmnK2S.7rYLgveMlQ0uvOwsJR2.oyNCjd7ksEt2K6SE7VK5toP1jnyQzQRj0HLapvScAECSKE/

#创建用户同时生成密码（centos和ubuntu通用）、
[root@centos8 data]# useradd -p `echo xiaoming | openssl passwd -6 -salt qweewq -stdin` user2
[root@centos8 data]# getent shadow user2
user2:$6$qweewq$Uy5hJssuD8tzMmnK2S.7rYLgveMlQ0uvOwsJR2.oyNCjd7ksEt2K6SE7VK5toP1jnyQzQRj0HLapvScAECSKE/:18719:0:99999:7:::
```

#### 2.3.4 openssl命令生成随机数

帮助：man sslrand

范例：

```bash
#采用base64编码输出9个字符随机数
[root@centos8 ~]# openssl rand -base64 9
mTLC8hWynv6M

#生成随机10位长度密码
[root@centos8 ~]# openssl rand -base64 9 | head -c10
cV5MLKUmHz[root@centos8 ~]# 
```

我们还可以使用随机数生成器生成随机数

随机数生成器：伪随机数字，利用键盘和鼠标，块设备中断生成随机数 /dev/random：仅从熵池返回随机数；随机数用尽，阻塞 /dev/urandom：从熵池返回随机数；随机数用尽，会利用软件生成伪随机数，非阻塞

范例：

```bash
#删除不是字母和数字的内容
[root@centos8 ~]# tr -dc '[:alnum:]' < /dev/random | head -c10
23u6MUama3
```

#### 2.3.5 openssl rsa命令实现非对称加密

openssl命令生成密钥对儿：man genrsa

生成私钥格式：

```bash
openssl genrsa -out /PATH/TO/PRIVATEKEY.FILE [-des3] [NUM_BITS,默认2048]
```

范例：

```bash
[root@centos8 data]# openssl genrsa -out app.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
....+++++
......+++++
e is 65537 (0x010001)

#将私钥使用des3加密,这样更安全不过会比较麻烦
[root@centos8 data]# openssl genrsa -out app2.key -des3 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
............+++++
..............................................................................+++++
e is 65537 (0x010001)
Enter pass phrase for app2.key:
Verifying - Enter pass phrase for app2.key:
```

提取公钥格式：

```bash
openssl rsa -in PRIVATEKEYFILE –pubout –out PUBLICKEYFILE
```

范例：解密之前生成的私钥

```bash
[root@centos8 data]# openssl rsa -in app2.key -pubout -out app2.key.pub
Enter pass phrase for app2.key:
writing RSA key
[root@centos8 data]# ls
app2.key  app2.key.pub  app.key  fstab  script
[root@centos8 data]# cat app2.key.pub 
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyIKRptwChdIgUiRFt3nV
BnyrbzqDqPcwECNYQeGHUivQCVYTxDny6WrZ8JscrLef1NL18Sa0l5rle8AuGg6N
EKB3OnWOfzXX9F8sc7GPMJTi4nK45SwnOR8k6KYYnIONDethTrbpNuLK0/O3EuYl
0Ez2/6QWu15bbjZUHxFXEsEbQQf3sKT+N68zJGrG8ZZOZjeBpoWGMvbucYks7aHt
bs4gq1uedSAf66f74CLMD/UmvXzYubWV1HZN7yqWJidfnnARRJ/shu7HlWavShun
X2YBDLXHLuOBHLqWNypa5U/emGpf3O2zFRjGKp/IdRq4e/jRL/qWOy/pXfzYY9Bd
nwIDAQAB
-----END PUBLIC KEY-----
```

### 2.4 建立私有CA实现证书申请颁发

openssl的配置文件：

```bash
#通常在创建根CA时会用到此文件
/etc/pki/tls/openssl.cnf
[root@centos8 ~]#cat /etc/pki/tls/openssl.cnf
......
[ CA_default ]

dir             = /etc/pki/CA           # Where everything is kept
certs           = $dir/certs            # Where the issued certs are kept
crl_dir         = $dir/crl              # Where the issued crl are kept
database        = $dir/index.txt        # database index file.
#unique_subject = no                    # Set to 'no' to allow creation of
                                        # several certs with same subject.
new_certs_dir   = $dir/newcerts         # default place for new certs.

certificate     = $dir/cacert.pem       # The CA certificate
serial          = $dir/serial           # The current serial number
crlnumber       = $dir/crlnumber        # the current crl number
                                        # must be commented out to leave a V1 CRL
crl             = $dir/crl.pem          # The current CRL
private_key     = $dir/private/cakey.pem# The private key

x509_extensions = usr_cert              # The extensions to add to the cert

# Extensions to add to a CRL. Note: Netscape communicator chokes on V2 CRLs
# so this is commented out by default to leave a V1 CRL.
# crlnumber must also be commented out to leave a V1 CRL.
# crl_extensions        = crl_ext

default_days    = 365                   # how long to certify for
default_crl_days= 30                    # how long before next CRL
default_md      = sha256                # use SHA-256 by default
preserve        = no                    # keep passed DN ordering

# A few difference way of specifying how similar the request should look
# For type CA, the listed attributes must be the same, and the optional
# and supplied fields are just that :-)
policy          = policy_match

# For the CA policy
[ policy_match ]
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional
```

三种策略：

- match：要求申请填写的信息跟CA设置信息必须一致
- optional：可有可无，跟CA设置信息可不一致
- supplied：必须填写这项申请信息

#### 2.4.1 创建私有根CA自签名证书

1、创建文件

```bash
[root@centos8 CA]# mkdir private
[root@centos8 CA]# touch private/cakey.pem
```

2、生成CA私钥

```bash
[root@centos8 pki]# mkdir CA;cd CA
[root@centos8 CA]# openssl genrsa -out private/cakey.pem 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
..................+++++
...............................................................................................+++++
e is 65537 (0x010001)
[root@centos8 CA]# ll private/cakey.pem 
-rw-r--r--. 1 root root 1679 Apr  4 07:20 private/cakey.pem
```

3、生成CA自签名证书

```bash
[root@centos8 CA]# openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -days 3650 -out /etc/pki/CA/cacert.pem
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:chengdu
Locality Name (eg, city) [Default City]:chengdu
Organization Name (eg, company) [Default Company Ltd]:tenxun
Organizational Unit Name (eg, section) []:jishu
Common Name (eg, your name or your server's hostname) []:
Email Address []:

#查看生成证书
[root@centos8 CA]# openssl x509 -in ./cacert.pem -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            06:af:b5:16:75:af:cb:41:b4:6c:65:fe:7c:fa:98:ab:85:e4:37:2c
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = CN, ST = chengdu, L = chengdu, O = tenxun, OU = jishu
        Validity
......
```

#### 2.4.2 申请证书并颁发证书

1、为需要使用证书的主机生成生成私钥

```bash
[root@centos8 CA]# openssl genrsa -out /data/test.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
........................................................................................................................+++++
...................................+++++
e is 65537 (0x010001)
```

2、为需要使用证书的主机生成证书申请文件

```bash
#注意：由于配置文件的选项国家省份组织这三个选项必须和CA配置一致
[root@centos8 CA]# openssl req -new -key /data/test.key -out /data/test.crs
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:CN
State or Province Name (full name) []:chengdu
Locality Name (eg, city) [Default City]:chengdu
Organization Name (eg, company) [Default Company Ltd]:tenxun
Organizational Unit Name (eg, section) []:renli
Common Name (eg, your name or your server's hostname) []:
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

3、在CA签署证书并将证书颁发给请求者

```bash
[root@centos8 CA]# openssl ca -in /data/test.crs -out /etc/pki/CA/newcerts/test.crt 
Using configuration from /etc/pki/tls/openssl.cnf
140065476847424:error:02001002:system library:fopen:No such file or directory:crypto/bio/bss_file.c:72:fopen('/etc/pki/CA/index.txt','r')
140065476847424:error:2006D080:BIO routines:BIO_new_file:no such file:crypto/bio/bss_file.c:79:
140674566915904:error:02001002:system library:fopen:No such file or directory:crypto/bio/bss_file.c:72:fopen('/etc/pki/CA/serial','r')
#注意：这里由于没有创建相关文件报错，所以先创建相关文件

[root@centos8 CA]# touch index.txt 
[root@centos8 CA]# touch serial
[root@centos8 CA]# echo 01 > /etc/pki/CA/serial

#再次颁发证书
[root@centos8 CA]# openssl ca -in /data/test.crs -out /etc/pki/CA/newcerts/test.crt
Using configuration from /etc/pki/tls/openssl.cnf
Check that the request matches the signature
......
1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
```

4、查看证书中的信息：

```bash
openssl x509 -in /PATH/FROM/CERT_FILE -noout   -text|issuer|subject|serial|dates
#查看指定编号的证书状态

openssl ca -status SERIAL
```

##### 2.4.3 吊销证书

在客户端获取要吊销的证书的serial

```bash
[root@centos8 CA]# openssl x509 -in /etc/pki/CA/newcerts/test.crt --noout -serial -subject
serial=01
subject=C = CN, ST = chengdu, O = tenxun, OU = renli
```

在CA上，根据客户提交的serial与subject信息，对比检验是否与index.txt文件中的信息一致，吊销证 书：

```bash
[root@centos8 CA]# openssl ca -revoke /etc/pki/CA/newcerts/01.pem 
Using configuration from /etc/pki/tls/openssl.cnf
Revoking Certificate 01.
Data Base Updated

#吊销完成证书发生改变的文件，其状态由v变为R
[root@centos8 CA]# cat /etc/pki/CA/index.txt
R	310401235120Z	210404000314Z	01	unknown	/C=CN/ST=chengdu/O=tenxun/OU=renli
```

和颁发证书一样此时也需要创建一个吊销证书列表文件

```bash
[root@centos8 CA]# echo 01 > /etc/pki/CA/crlnumber
```

更新证书吊销列表

```bash
[root@centos8 CA]# openssl ca -gencrl -out /etc/pki/CA/crl.pem
Using configuration from /etc/pki/tls/openssl.cnf
```

查看

```bash
[root@centos8 CA]# openssl crl -in /etc/pki/CA/crl.pem -noout -text
Certificate Revocation List (CRL):
        Version 2 (0x1)
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = CN, ST = chengdu, L = chengdu, O = tenxun, OU = jishu
        Last Update: Apr  4 00:08:48 2021 GMT
        Next Update: May  4 00:08:48 2021 GMT
......
```

#### 2.4.4 CentOS 7 创建自签名证书

在centos7中可以利用/etc/pki/tls/certs文件下的makefile脚本来帮助我们完成证书吊销和颁发

```bash
[root@centos7 ~]#cd /etc/pki/tls/certs
[root@centos7 certs]#cat Makefile
UTF8 := $(shell locale -c LC_CTYPE -k | grep -q charmap.*UTF-8 && echo -utf8)
DAYS=365
KEYLEN=2048
[root@centos7 certs]#make app.crt
```

由于centos8没有这个文件，因此可以将makefile文件复制到centos8中，也能实现脚本颁发证书

## 3 ssh服务

### 3.1 服务介绍

ssh: secure shell, protocol, 22/tcp, 安全的远程登录，实现加密通信，代替传统的 telnet 协议

具体的软件实现：

- OpenSSH：ssh协议的开源实现，CentOS 默认安装
- dropbear：另一个ssh协议的开源项目的实现

**ssh首次连接时公钥交换**

https://www.ssh.com/ssh/protocol/

**ssh通讯过程中加密原理**

### 3.2 openssh 服务

OpenSSH是SSH （Secure SHell） 协议的免费开源实现，一般在各种Linux版本中会默认安装，基于 C/S结构

openssh服务端相关文件：

- 服务端：/usr/sbin/sshd
- unit文件：/usr/lib/systemd/system/sshd.service

openssh客户端相关文件：

- /usr/bin/ssh
- /etc/ssh/ssh_config
- /etc/ssh中相关密钥文件
- 用户家目录中公钥文件

#### 3.2.1 客户端登陆方式和原理

ssh服务登陆常用验证方式

- 用户和口令
- 基于密钥

**基于用户和口令登陆验证**

**基于密钥登陆验证**

#### 3.2.2 ssh客户端命令

ssh命令是ssh客户端，允许实现对远程系统经验证地加密安全访问

格式：

```bash
ssh [user@]host [COMMAND]
ssh [-l user] host [COMMAND]
```

常见选项：

- -p 指定远程服务器监听端口
- -b 指定连接的源IP，可用于本机多ip情况
- -v 调试模式
- -c 压缩方式传输数据
- -x 支持x11转发，基于x11可实现本地显示远程图形界面
- -t 强制伪tty分配，用于多主机跳转情况
- -o option 可用于指定配置文件中设置
- -i 指定私钥文件路径

范例：ssh连接时不询问是否连接

```bash
[root@centos7 ~]#sed -i.bak '/StrictHostKeyChecking/s/.*/StrictHostKeyChecking
no/' /etc/ssh/ssh_config
```

范例：一条命令实现多主机跳转至目标主机

```bash
[root@centos8 ~]#ssh -t 10.0.0.8 ssh -t 10.0.0.7 ssh 10.0.0.6
root@10.0.0.8's password:
root@10.0.0.7's password:
root@10.0.0.6's password:
Last login: Fri May 22 09:10:28 2020 from 10.0.0.7
```

范例：远程执行命令

```bash
#主机后面直接跟命令
[root@localhost ~]# ssh root@192.168.1.140 hostname -I
The authenticity of host '192.168.1.140 (192.168.1.140)' can't be established.
ECDSA key fingerprint is SHA256:Ft+oMSnzLdMmXE3nOIRaRvrstvDiRj35kInIufWMrPk.
ECDSA key fingerprint is MD5:c2:94:4d:01:29:57:23:eb:d2:2f:da:a1:c5:87:04:c1.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.1.140' (ECDSA) to the list of known hosts.
root@192.168.1.140's password: 
192.168.1.140 

#输入重定向方式执行命令
[root@centos8 ~]#ssh 10.0.0.18 /bin/bash < test.sh
root@10.0.0.18's password:
10.0.0.18 
```

##### 3.2.3 基于密钥登陆的实现

1、客户端生成密钥对

```bash
ssh-keygen -t rsa [-P 'password'] [-f “~/.ssh/id_rsa"]
```

选项：

- -p 对密钥进行加密
- -f 指定密钥文件

还可以对私钥重设口令

```bash
ssh-keygen –p 
```

2、把公钥文件传输至远程服务器对应用户的家目录中

```bash
ssh-copy-id [-i [identity_file]] [user@]host
```

3、使用ssh登陆

```bash
ssh [user@]host [COMMAND]
```

注意：当我们使用基于密钥方式登陆服务端时，服务端是通过客户端私钥进行验证，因此任何一个客户端拿到私钥都可以使用此方式远程登陆服务端。所以为了保证私钥安全，我们可以对私钥进行加密。

**验证代理（authentication agent）**

保密解密后的密钥，口令就只需要输入一次，从而避免了交互式登陆

代理启用方式：

```bash
#启用代理
ssh-agent bash
#钥匙通过命令添加给代理
ssh-add
```

##### 3.2.4 其他ssh客户端工具

**scp命令**

格式：

```bash
scp [options] SRC... DEST/

scp [options] [user@]host:/sourcefile /destpath
scp [options] /sourcefile [user@]host:/destpath
```

常用选项：

- -C 压缩数据流
- -p 保持原文件的属性信息
- -r 递归复制
- -q 静默模式
- -P PORT 指明remote host的监听的端口

**rsync 命令**

rsync工具可以基于ssh和rsync协议实现高效率的远程系统之间复制文件，使用安全的shell连接做为传 输方式，比scp更快，基于增量数据同步，即只复制两方不同的文件，此工具来自于rsync包

注意：通信两端主机都需要安装 rsync软件

格式：

```bash
rsync  -av /etc server1:/tmp #复制目录和目录下文件
rsync  -av /etc/ server1:/tmp #只复制目录下文件
```

常用选项：

- -v 显示详细过程
- -r 递归赋值目录树
- -p 保留权限
- -t 保留修改时间戳
- -g 保留组信息
- -o 保留所有者信息
- -l 将软链接文件本身进行复制
- -L 将软链接文件指向的文件复制
- -u 如果接收者的文件比发送者的文件较新，将忽略同步
- -z 压缩，节约网络带宽
- -a 存档，相当于–rlptgoD，但不保留ACL（-A）和SELinux属性（-X）
- --delete 源数据删除，目标数据也自动同步删除

**sftp命令**

格式：

```bash
sftp [user@]host
sftp> help
```

##### 3.3.5 ssh高级应用

**本地端口转发**

原理图：

简单来说是将要转发的数据封装在ssh协议中发送，提高了安全性

SSH 端口转发能够提供两大功能：

- 加密 SSH Client 端至 SSH Server 端之间的通讯数据
- 突破防火墙的限制完成一些之前无法建立的 TCP 连接

格式：

```bash
#在客户端执行
ssh -L localport:remotehost:remotehostport   sshserver
```

选项：

- -f 后台启用
- -N 不打开远程shell
- -g 启用网关功能

范例：

```bash
#客户端执行命令建立通道
ssh –L 9527:telnetsrv:23 -Nfg sshsrv

#访问客户端本机端口,ssh服务器代理转发数据
telnet 127.0.0.1 9527
```

##### 3.3.6 ssh服务器配置选项

配置文件路径：/etc/ssh/sshd_config

配置文件帮助文档：man 5 sshd_config

常用配置设置：

```bash
Port
ListenAddress ip
LoginGraceTime 2m
PermitRootLogin yes 				#默认ubuntu不允许root远程ssh登录
StrictModes yes   				#检查.ssh/文件的所有者，权限等
MaxAuthTries   6     			#身份验证的允许失败的最大次数
MaxSessions  10         		#同一个连接最大会话
PubkeyAuthentication yes     #基于key验证
PermitEmptyPasswords no      #空密码连接
PasswordAuthentication yes   #基于用户名和密码连接
GatewayPorts no
ClientAliveInterval 10 			#单位:秒
ClientAliveCountMax 3 			#默认3
UseDNS yes 							#提高速度可改为no
GSSAPIAuthentication yes 		#提高速度可改为no
MaxStartups    					#未认证连接最大值，默认值10
Banner /path/file

#以下可以限制可登录用户的办法：
AllowUsers user1 user2 user3
DenyUsers
AllowGroups
DenyGroups
```

范例：提升ssh连接速度

```bash
vim /etc/ssh/sshd_config
UseDNS no
GSSAPIAuthentication no

systemctl restart sshd
```

ssh服务建议配置策略：

- 建议使用非默认端口
- 禁止使用protocol version 1
- 限制可登录用户
- 设定空闲会话超时时长
- 利用防火墙设置ssh访问策略
- 仅监听特定的IP地址
- 基于口令认证时，使用强密码策略，比如：tr -dc A-Za-z0-9_ < /dev/urandom | head -c 12| xargs
- 使用基于密钥的认证
- 禁止使用空密码
- 禁止root用户直接登录
- 限制ssh的访问频度和并发在线数
- 经常分析日志

### 3.3 ssh 其它相关工具

##### 3.4.1 自动登录ssh工具sshpass

EPEL源提供，ssh登陆不能在命令行中指定密码。sshpass的出现，解决了这一问题。同样可以解决非交互式登陆的还有expect，只不过expect语法不友好，使用sshpass可以更为方便实现非交互式登陆。

格式：

```bash
sshpass [option] command parameters
```

常用选项：

- -p password #后跟密码它允许你用 -p 参数指定明文密码，然后直接登录远程服务器
- -f filename #后跟保存密码的文件名，密码是文件内容的第一行。
- -e #将环境变量SSHPASS作为密码

范例：

```bash
[root@centos8 ~]#sshpass -p 123456 ssh -o StrictHostKeyChecking=no 10.0.0.6
hostname -I
```

范例：实现批量下发密钥

```bash
[root@server1 ssh]# cat ssh_key 
#!/bin/bash
HOSTS="
172.16.60.110
172.16.60.114
"
PASS=123321
ssh-keygen -P "" -f /root/.ssh/id_rsa &> /dev/null
rpm -q sshpass &> /dev/null || yum -y install sshpass &> /dev/null
for i in $HOSTS;do
	{
		sshpass -p $PASS ssh-copy-id -o StrictHostKeyChecking=no -i /root/.ssh/id_rsa.pub $i &> /dev/null
	}&
done
wait
```

## 4 文件完整性检查

AIDE(Advanced Intrusion Detection Environment高级入侵检测环境)是一个入侵检测工具，主要用途 是检查文件的完整性，审计计算机上的那些文件被更改过了

AIDE能够构造一个指定文件的数据库，它使用aide.conf作为其配置文件。AIDE数据库能够保存文件的 各种属性，包括：权限(permission)、索引节点序号(inode number)、所属用户(user)、所属用户组 (group)、文件大小、最后修改时间(mtime)、创建时间(ctime)、最后访问时间(atime)、增加的大小以 及连接数。AIDE还能够使用下列算法：sha1、md5、rmd160、tiger，以密文形式建立每个文件的校 验码或散列号

通常来说数据库不应该保存那些经常变动的文件信息，例如日志文件等

配置文件

```bash
vim /etc/aide.conf 
```

范例：

```bash
#定义监控项权限+索引节点+链接数+用户+组+大小+最后一次修改时间+创建时间+md5校验值
R=p+i+n+u+g+s+m+c+md5
NORMAL = R+rmd60+sha256
/data/test.txt R
/bin/ps R+a
/usr/bin/crontab R+a
/etc   PERMS
!/etc/mtab   #“!”表示忽略这个文件的检查
```

初始化默认的AIDE的库：

```bash
/usr/local/bin/aide -i | --init
```

生成检查数据库（建议初始数据库存放到安全的地方）

```bash
cd /var/lib/aide
mv aide.db.new.gz aide.db.gz
```

检测

```bash
/usr/local/bin/aide -C | --check
```

更新数据库

```bash
aide -u | --update
```

## 5 sudo命令

### 5.1 sudo介绍

sudo 即superuser do，允许系统管理员让普通用户执行一些或者全部的root命令的一个工具，如 halt，reboot，su等等。这样不仅减少了root用户的登录 和管理时间，同样也提高了安全性

### 5.2 sudo 组成

配置文件：

/etc/sudo.conf,一般很少配置此文件，使用man 5 sudo.conf可以查看文件说明

授权规则配置文件：

/etc/sudoers文件和/etc/sudoers.d目录，我们可以将sudo配置写入/etc/sudoers文件中，也可以自定义一个sudo文件至/etc/sudoers.d目录

安全编辑授权规则文件和语法检查工具

/usr/sbin/visudo

日志文件：

/var/log/secure

范例：

```bash
#检查语法
[root@server1 .ssh]visudo -c

#检查指定配置文件语法
[root@server1 .ssh]visudo -f /etc/sudoers.d/test

#设置默认编辑器为vim
[root@server1 .ssh]# cat ~/.bashrc 
export EDITOR="vim"
```

### 5.3 sudo命令

格式：

```bash
sudo [-u user] COMMAND
```

常用选项：

- -u user 默认为root用户
- -l 列出用户在主机上可用的和被禁止的命令
- -v 延长密码的有效期限5分钟

### 5.4 sudo 授权规则配置

注意：配置文件中支持使用通配符 glob

配置文件：/etc/sudoers, /etc/sudoers.d/

suduoer授权规则格式：

```bash
用户 登入主机=(代表用户) 命令
user host=(runas) command

user: 运行命令者的身份
host: 通过哪些主机
(runas)：以哪个用户的身份
command: 运行哪些命令

User和runas形式:
	username
 	#uid
 	%group_name
 	%#gid
 	user_alias|runas_alias	
host:
   ip或hostname
   network(/netmask)
   host_alias
command:
   command name
   directory
   sudoedit
   Cmnd_Alias
```

sudoers的可自定义别名：

- User_Alias
- Runas_Alias
- Host_Alias
- Cmnd_Alias

别名格式：

```bash
[A-Z]([A-Z][0-9]_)*
```

别名定义：

```bash
Alias_Type NAME1 = item1,item2,item3 : NAME2 = item4, item5
```

范例：

```bash
#允许以tom和jerry的身份执行所有命令。默认是以tom身份执行
Defaults:user1 runas_default=tom
user1 ALL=(tom,jerry) ALL
```

## 6 PAM认证机制

PAM：Pluggable Authentication Modules，插件式的验证模块，Sun公司于1995 年开发的一种与认 证相关的通用框架机制。它提供API给程序开发人员和系统管理人员，这样用户无需再自己开发一套认证模块。

### 6.1 PAM工作原理

PAM认证一般遵循这样的顺序：Service(服务)→PAM(配置文件)→pam_*.so PAM认证首先要确定那一项服务，然后加载相应的PAM的配置文件(位于/etc/pam.d下)，最后调用认证 文件(位于/lib64/security下)进行安全认证

认证过程示例：

```bash
1.使用者执行/usr/bin/passwd 程序，并输入密码
2.passwd开始调用PAM模块，PAM模块会搜寻passwd程序的PAM相关设置文件，这个设置文件一般是
在/etc/pam.d/里边的与程序同名的文件，即PAM会搜寻/etc/pam.d/passwd此设置文件
3.经由/etc/pam.d/passwd设定文件的数据，取用PAM所提供的相关模块来进行验证
4.将验证结果回传给passwd这个程序，而passwd这个程序会根据PAM回传的结果决定下一个动作（重新输入
密码或者通过验证）
```

### 6.2 PAM相关文件

- 模块文件目录：/lib64/security/*.so

- 特定模块相关的设置文件：/etc/security/

- 应用程序调用PAM模块的配置文件

  主配置文件：/etc/pam.conf，默认不存在，一般不使用主配置

  为每种应用模块提供一个专用的配置文件：/etc/pam.d/APP_NAME

  注意：如/etc/pam.d存在，/etc/pam.conf将失效

范例：查看程序是否支持PAM

```bash
[root@centos8 ~]#ldd `which sshd` |grep libpam
 libpam.so.0 => /lib64/libpam.so.0 (0x00007fea8e70d000)
[root@centos8 ~]#ldd `which passwd` |grep pam
 libpam.so.0 => /lib64/libpam.so.0 (0x00007f045b805000)
 libpam_misc.so.0 => /lib64/libpam_misc.so.0 (0x00007f045b601000)
```

### 6.3 PAM配置文件

通用配置文件/etc/pam.conf格式

```bash
application type control module-path arguments
```

专用配置文件/etc/pam.d/ 格式

```bash
type control module-path arguments

application：指服务名，如：telnet、login、ftp等，服务名字“OTHER”代表所有没有在该文件中明确
配置的其它服务
type：指模块类型，即功能
control ：PAM库该如何处理与该服务相关的PAM模块的成功或失败情况，一个关健词实现
module-path： 用来指明本模块对应的程序文件的路径名
Arguments： 用来传递给该模块的参数
```

帮助可以参看官方文档

官方在线文档：http://www.linux-pam.org/Linux-PAM-html/ 官方离线文档：http://www.linux-pam.org/documentation/

也可以使用man帮助

```bash
man 模块名 如：man 8 rootok
```

## 7 时间同步服务

加密和安全当前都离不开时间的同步，否则各种网络服务可能不能正常运行

### 7.1 时间同步服务

多主机协作工作时，各个主机的时间同步很重要，时间不一致会造成很多重要应用的故障，如：加密协 议，日志，集群等， 利用NTP（Network Time Protocol） 协议使网络中的各个计算机时间达到同步。 目前NTP协议属于运维基础架构中必备的基本服务之一

时间同步软件：

- ntp
- chrony

详细ntp信息可参看官网：[http://www.ntp.org](http://www.ntp.org/)

### 7.2 chrony

##### 7.2.1 chrony介绍

由于chrony的优势，现在chrony成为较受欢迎的时间同步软件

chrony官网：[https://chrony.tuxfamily.org](https://chrony.tuxfamily.org/)

chrony官方文档：https://chrony.tuxfamily.org/documentation.html

##### 7.2.2 chrony 文件组成

两个主要程序：chronyd和chronyc

- chronyd：后台运行的守护进程，用于调整内核中运行的系统时钟和时钟服务器同步。它确定计算 机增减时间的比率，并对此进行补偿
- chronyc：命令行用户工具，用于监控性能并进行多样化的配置。它可以在chronyd实例控制的计 算机上工作，也可在一台不同的远程计算机上工作

服务unit 文件： /usr/lib/systemd/system/chronyd.service

监听端口： 323/udp，123/udp

配置文件： /etc/chrony.conf

##### 7.2.3 配置文件chrony.conf

```bash
server - 可用于时钟服务器，iburst 选项当服务器可达时，发送一个八个数据包而不是通常的一个数据
包。 包间隔通常为2秒,可加快初始同步速度
driftfile - 根据实际时间计算出计算机增减时间的比率，将它记录到一个文件中，会在重启后为系统时钟
作出补偿
rtcsync - 启用内核模式，系统时间每11分钟会拷贝到实时时钟（RTC）
allow / deny - 指定一台主机、子网，或者网络以允许或拒绝访问本服务器
cmdallow / cmddeny - 可以指定哪台主机可以通过chronyd使用控制命令
bindcmdaddress - 允许chronyd监听哪个接口来接收由chronyc执行的命令
makestep - 通常chronyd将根据需求通过减慢或加速时钟，使得系统逐步纠正所有时间偏差。在某些特定
情况下，系统时钟可能会漂移过快，导致该调整过程消耗很长的时间来纠正系统时钟。该指令强制chronyd在
调整期大于某个阀值时调整系统时钟
local stratum 10  - 即使server指令中时间服务器不可用，也允许将本地时间作为标准时间授时给其
它客户端
```

##### 7.2.4 chronyc命令

```bash
help 命令可以查看更多chronyc的交互命令
accheck 检查是否对特定主机可访问当前服务器
activity 显示有多少NTP源在线/离线
sources [-v]   显示当前时间源的同步信息
sourcestats [-v]显示当前时间源的同步统计信息
add server 手动添加一台新的NTP服务器
clients 报告已访问本服务器的客户端列表
delete 手动移除NTP服务器或对等服务器
settime 手动设置守护进程时间
tracking 显示系统时间信息
```

##### 7.2.5 公共NTP服务

- 阿里云公共NTP服务器

  Unix/linux类：ntp.aliyun.com，ntp1-7.aliyun.com windows类： time.pool.aliyun.com

- 大学ntp服务

  s1a.time.edu.cn 北京邮电大学

  s1b.time.edu.cn 清华大学

  s1c.time.edu.cn 北京大学

- 国家授时中心服务器：210.72.145.44

练习

1、PAM和google模块实现ssh双因子安全验证。

2、使用chrony实现内网时间同步（一台node1从外网同步时间，其余机器从node1同步时间）。

```bash
1、配置同步服务器
#安装软件包
[root@server1 .ssh]# rpm -qa | grep chrony
chrony-3.4-1.el7.x86_64

#修改配置文件
[root@server1 .ssh]# cat /etc/chrony.conf
server ntp.aliyun.com iburst
allow  0.0.0.0/24
local stratum 10

#重启服务
[root@server1 .ssh]# systemctl restart chronyd
#查看是否连上同步服务器
[root@server1 .ssh]# chronyc sources -v
210 Number of sources = 4

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* 203.107.6.88                  2   6    77     2  -1417us[+4237us] +/-   52ms

2、配置客户端
#修改配置文件服务器
[root@localhost ~]# grep server /etc/chrony.conf 
# Use public servers from the pool.ntp.org project.
server 172.16.60.238 iburst
#重启服务
[root@localhost ~]# systemctl restart chronyd
#查看是否连接成功
[root@localhost ~]# chronyc sources -v
210 Number of sources = 1

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* 172.16.60.238                 3   6    17     3  +7044ns[ +108us] +/-   50ms
```

# 6.系统自动部署

## 1 基于centos8实现PXE自动安装系统

红帽官方文档：

https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/installation_guide/chap-installation-server-setup#sect-network-boot-setup-bios

Syslinux参考站点：

https://wiki.syslinux.org/wiki/index.php?title=PXELINUX#Description

https://wiki.archlinux.org/index.php/Syslinux_(简体中文)

### 1.1 安装前准备

关闭防火墙和SELINUX，DHCP服务器静态IP

网络要求：关闭Vmware软件中的DHCP服务，基于NAT模式

主机分配：将所有服务安装在同一台服务器中

实现centos8的自动安装

### 1.2 安装相关软件包

```bash
#安装软件包
[root@centos8 ~]# yum -y install dhcp-server tftp-server httpd syslinux-nonlinux

#启动服务
[root@centos8 ~]# systemctl enable --now httpd tftp

#查看是否启动成功
[root@centos8 ~]# ss -ntul
Netid      State       Recv-Q      Send-Q            Local Address:Port             Peer Address:Port      
udp        UNCONN      0           0                             *:69               *:*         
tcp        LISTEN      0           128                     0.0.0.0:22             0.0.0.0:*     
tcp        LISTEN      0           128                           *:80               *:*         
tcp        LISTEN      0           128                        [::]:22               [::]:*
```

### 1.3 配置DHCP服务

```bash
#默认/etc下dhcp文件是空的，所以先拷贝范例配置文件便于配置
[root@centos8 ~]# cp /usr/share/doc/dhcp-server/dhcpd.conf.example /etc/dhcp/dhcpd.conf

#配置DHCP
[root@centos8 ~]# ip a
1: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:ed:28:61 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.100/24 scope global ens160
       valid_lft forever preferred_lft forever

[root@centos8 ~]# vim /etc/dhcp/dhcpd.conf 	#注意不要丢失尾部分号，否则启动不了服务
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;

default-lease-time 600;
max-lease-time 7200;
log-facility local7;

subnet 10.0.0.0 netmask 255.255.255.0 {
 range 10.0.0.1 10.0.0.200;
 option routers 10.0.0.1;
 next-server 10.0.0.100;
 filename "pxelinux.0";
}

#启动dhcp服务
[root@centos8 ~]# systemctl start dhcpd
```

### 1.4 准备yum 源和相关目录

```bash
#创建文件夹
[root@centos8 ~]# mkdir -pv /var/www/html/centos/8/os/x86_64/
mkdir: created directory '/var/www/html/centos'
mkdir: created directory '/var/www/html/centos/8'
mkdir: created directory '/var/www/html/centos/8/os'
mkdir: created directory '/var/www/html/centos/8/os/x86_64/'
[root@centos8 ~]# ll /var/www/html/centos/8/os/x86_64/ -d
drwxr-xr-x. 2 root root 6 Apr 29 01:52 /var/www/html/centos/8/os/x86_64/

#挂载本地光盘
[root@centos8 ~]# mount /dev/sr0 /var/www/html/centos/8/os/x86_64/
mount: /var/www/html/centos/8/os/x86_64: WARNING: device write-protected, mounted read-only.

#这时我们可以访问http://10.0.0.100/centos/8/os/x86_64/来验证是否成功
```

### 1.5 准备PXE启动相关文件

```bash
#创建文件夹
[root@centos8 ~]# mkdir /var/lib/tftpboot/centos8

#复制内核文件至tftp目录
[root@centos8 ~]# cp /var/www/html/centos/8/os/x86_64/isolinux/{vmlinuz,initrd.img} /var/lib/tftpboot/centos8/

#复制syslinux相关文件至tftp
[root@centos8 ~]# cp /usr/share/syslinux/{pxelinux.0,ldlinux.c32,menu.c32,libcom32.c32,libutil.c32} /var/lib/tftpboot/
[root@centos8 ~]# ll /var/lib/tftpboot/
total 160
drwxr-xr-x. 2 root root     39 Apr 30 00:02 centos8
-rw-r--r--. 1 root root 116096 Apr 30 01:05 ldlinux.c32
-rw-r--r--. 1 root root  42791 Apr 30 01:05 pxelinux.0

#生成菜单文件
[root@centos8 ~]# mkdir /var/lib/tftpboot/pxelinux.cfg/
[root@centos8 ~]# cp /var/www/html/centos/8/os/x86_64/isolinux/isolinux.cfg /var/lib/tftpboot/pxelinux.cfg/default
```

### 1.6 准备kickstart文件

```bash
[root@centos8 ~]# mkdir /var/www/html/ks
[root@centos8 ~]# cat /var/www/html/ks/centos8.cfg 
ignoredisk --only-use=sda
zerombr
text
reboot
clearpart --all --initlabel
selinux --disabled
firewall --disabled
url --url=http://10.0.0.100/centos/8/os/x86_64/
keyboard --vckeymap=us --xlayouts='us'
lang en_US.UTF-8
network  --bootproto=dhcp --device=ens160 --ipv6=auto --activate
network  --hostname=centos8.magedu.com
rootpw --iscrypted $6$18lCu5Jb7ncPFxK2$xu15Q3igR6OmU71EiUxuzEv7eic7sYdSmhE7kuRfZ8VfqwCrm/0AbojEgvIz0KXg4HntKbufBmnx/dgqsSvOY.
firstboot --enable
skipx
services --disabled="chronyd"
timezone Asia/Shanghai --isUtc --nontp
part / --fstype="xfs" --ondisk=sda --size=102400
part /data --fstype="xfs" --ondisk=sda --size=51200
part swap --fstype="swap" --ondisk=sda --size=2048
part /boot --fstype="ext4" --ondisk=sda --size=1024
%packages
@^minimal-environment
kexec-tools
%end
%addon com_redhat_kdump --enable --reserve-mb='auto'
%end
%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end
%post
useradd test1
%end
```

### 1.7 配置启动菜单文件

```bash
[root@centos8 ~]# vim /var/lib/tftpboot/pxelinux.cfg/default
[root@centos8 ~]# cat /var/lib/tftpboot/pxelinux.cfg/default 
default menu.c32
timeout 600

menu title CentOS Linux 8

label linux8
  menu label ^Install CentOS Linux 8
  kernel centos8/vmlinuz
  append initrd=centos8/initrd.img ks=http://10.0.0.100/ks/centos8.cfg

label manual
  menu label ^Manual Install CentOS Linux 8.0
  kernel centos8/vmlinuz
  append initrd=centos8/initrd.img
 inst.repo=http://10.0.0.100/centos/8/os/x86_64/ 

label rescue
  menu label ^Rescue a CentOS Linux system 8
  kernel centos8/vmlinuz
  append initrd=centos8/initrd.img
 inst.repo=http://10.0.0.100/centos/8/os/x86_64/ rescue
  
label local
  menu default
  menu label Boot from ^local drive
  localboot 0xffff
```

### 1.8 测试客户端基于PXE实现自动安装

菜单选择界面

[![image](https://img2020.cnblogs.com/blog/2222624/202104/2222624-20210430153828733-415348424.png)](https://img2020.cnblogs.com/blog/2222624/202104/2222624-20210430153828733-415348424.png)

系统自动安装完成后查看是否成功创建test1用户
[![image](https://img2020.cnblogs.com/blog/2222624/202104/2222624-20210430153845855-53538110.jpg)](https://img2020.cnblogs.com/blog/2222624/202104/2222624-20210430153845855-53538110.jpg)

# 7.DNS服务

## 1 名字解析介绍和DNS

### 1.1 DNS是什么

DNS：Domain Name System 域名系统,应用层协议,是互联网的一项服务。主要作用是根据域名查出IP地址，当然也可以通过IP地址查询域名。

### 1.2 DNS域名层次结构

对于域名其层级结构如下

```bash
主机名.次级域名.顶级域.根域名
```

例如：www.example.com[.root](注意这里最后还有一个.root，它代表根域名，由于.root对于所有域名都一样，所以平时一般会省略),其中com为顶级域名，example为二级域名，www为三级域名。

根域名服务器（Root DNS Server）:管理顶级域名服务器返回顶级域名服务器IP，如com、cn

顶级域名服务器：返回二级域IP

### 1.3 DNS查询过程

DNS查询类型

- 递归查询：最终结果，负责到底
- 迭代查询：最好结果，不负责到底

DNS请求过程

```bash
Client -->hosts文件 --> Client DNS Service Local Cache --> DNS Server (recursion
递归) --> DNS Server Cache -->DNS iteration(迭代) --> 根--> 顶级域名DNS-->二级域名
DNS… 
```

这里假设访问www.qq.com

- 首先会在本地查找host文件中查找，若没有则会去本地DNS缓存中去查找
- 接着DNS客户端设置使用的DNS服务器，它负责全权处理客户端的DNS查询请求，直到返回最终结果。
- DNS服务器首先检查自身缓存，如果存在记录则直接返回结果。
- 如果记录老化或不存在，DNS服务器向根域名服务器发送查询报文"query www.qq.com"，根域名服务器返回顶级域.com的顶级域名服务器地址。
- DNS服务器向 .com域的顶级域名服务器发送查询报文"query www.qq.com"，得到二级域qq.com
- DNS服务器向 .qq.com 域的权威域名服务器发送查询报文"query www.qq.com",得到主机www的A记录，A是address的缩写。存入自身缓存并返回给客户端

### 1.4 DNS的记录类型

域名与IP之间的对应关系，称为”记录”（record）。根据使用场景，”记录”可以分成不同的类型（type）

常见的DNS记录类型如下：

- A：IPV4地址记录，传回一个32位的IPv4地址，最常用于映射主机名称到IP地址
- AAAA：IPV6地址记录，传回一个128位的IPv6地址，最常用于映射主机名称到IP地址
- SOA：权威记录的起始，指定有关DNS区域的权威性信息，包含主要名称服务器、域名管理员的电邮地址、域名的流水式编号、和几个有关刷新区域的定时器
- PTR：逆向查询记录（Pointer Record），只用于从IP地址查询域名
- NS：Name Server，专用于标明当前区域的DNS服务器
- MX：Mail eXchanger，邮件交换器
- CNAME ：规范名称记录， 一个主机名字的别名：域名系统将会继续尝试查找新的名字。

更多对应列表参看：https://zh.wikipedia.org/wiki/DNS记录类型列表

## 2 DNS软件bind

DNS服务器软件包括：bind，powerdns，unbound，coredns，其中最为广泛使用的就是bind

官方文档页：https://bind9.readthedocs.io/en/latest/introduction.html

文件定义参照标准：https://tools.ietf.org/html/rfc1035.html

红帽官方配置参考：https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/ch-DNS_Servers#s2-dns-introduction-zones

### 2.1 BIND相关程序包

bind：服务器

bind-utils: 客户端

bind-libs：相关库

bind-chroot: 安全包，将dns相关文件放至 /var/named/chroot/

### 2.2 BIND包相关文件

使用`yum -y install bind`查看bind相关文件，这里介绍几个主要文件

```bash
[root@centos8 yum.repos.d]# rpm -ql bind
/usr/sbin/named	#BIND主程序
/etc/rc.d/init.d/named	#服务脚本
/usr/lib/systemd/system/named.service	#Unit名称
/etc/named.conf	#主配置文件
/etc/named.rfc1912.zones	#主配置文件
/etc/rndc.key	#主配置文件
/usr/sbin/rndc	#remote name domain controller，默认与bind安装在同一主机，且只能通过127.0.0.1连接named进程，提供辅助性的管理功能；953/tcp
/var/named/ZONE_NAME.ZONE	#解析库文件
注意：
 (1) 一台物理服务器可同时为多个区域提供解析
 (2) 必须要有根区域文件；named.ca
 (3) 应该有两个（如果包括ipv6的，应该更多）实现localhost和本地回环地址的解析库
```

### 2.3 配置文件

#### 2.3.1 主配置文件/etc/named.conf

```bash
#服务端一般会对option中监听地址进行配置，注释掉listen-on port默认监听所有地址，该文件格式支持C和C++注释风格，所以可以使用//进行注释
[root@centos8 yum.repos.d]#vim /usr/share/doc/bind/sample/etc/named.conf
options
{
        // Put files that named is allowed to write in the data/ directory:
        directory               "/var/named";           // "Working" directory
        dump-file               "data/cache_dump.db";
        statistics-file         "data/named_stats.txt";
        memstatistics-file      "data/named_mem_stats.txt";
        secroots-file           "data/named.secroots";
        recursing-file          "data/named.recursing";


        /*
          Specify listenning interfaces. You can use list of addresses (';' is
          delimiter) or keywords "any"/"none"
        */
        //listen-on port 53     { any; };		允许哪些主机访问
        //	allow-query     { localhost; };	允许哪些主机查询
......
```

#### 2.3.2 区域声明文件/etc/named.rfc1912.zones

```bash
#声明区域zone，也可以在/etc/named.conf中声明。如果安装了 caching-nameserver包，则该文件是默认的配置文件
[root@centos8 yum.repos.d]# vim /usr/share/doc/bind/sample/etc/named.rfc1912.zones
zone "localhost.localdomain" IN {
        type master;
        file "named.localhost";
        allow-update { none; };
};

zone "localhost" IN {
        type master;
        file "named.localhost";
        allow-update { none; };
};
```

#### 2.3.3 区域数据库/var/named

```bash
#DNS服务器工作目录，可以将区域数据库文件存放于此文件夹中
[root@centos8 yum.repos.d]# ll /var/named/
total 16
drwxrwx---. 2 named named    6 Mar  1 23:20 data
drwxrwx---. 2 named named    6 Mar  1 23:20 dynamic
-rw-r-----. 1 root  named 2253 Mar  1 23:21 named.ca
-rw-r-----. 1 root  named  152 Mar  1 23:21 named.empty
-rw-r-----. 1 root  named  152 Mar  1 23:21 named.localhost	#数据库配置范例文件
-rw-r-----. 1 root  named  168 Mar  1 23:21 named.loopback
drwxrwx---. 2 named named    6 Mar  1 23:20 slaves

#数据库配置文件范例
[root@centos8 yum.repos.d]# vim /usr/share/doc/bind/sample/var/named/named.localhost
$TTL 1D
@       IN SOA  @ rname.invalid. (
                                      0       ; serial   #版本号，从服务器更新时会对比版本号
                                      1D      ; refresh	#从服务器多久同步一次数据
                                      1H      ; retry		#同步失败后多久再次进行同步
                                  1W      ; expire	#从服务器多久无法与主服务器同步后，停止对外提供服务
                                      3H )    ; minimum	#BIND9中代表未查询到结果的域名缓存时间
        NS      @
        A       127.0.0.1
        AAAA    ::1
```

## 3 实现主从DNS服务器

### 3 .1 实验准备

实验前提

```bash
关闭SElinux
关闭防火墙
时间同步
```

环境准备

```bash
需要四台主机
DNS主服务器：10.0.0.5
DNS从服务器:10.0.0.4
web服务器：10.0.0.5
DNS客户端：10.0.0.2
```

### 3.2 实验步骤

#### 3.2.1 主DNS服务器配置

安装服务端和客户端软件

```bash
#bind-utils是DNS客户端工具，这里主要用来检查区域文件语法
[root@centos8 ~]# yum -y install bind bind-utils	
```

修改配置文件

```bash
#主配置文件中只需修改下面两行,以确保监听的不仅仅是本地端口
[root@centos8 ~]# grep -e "listen-on port" -e "allow-query" /etc/named.conf 
//	listen-on port 53 { 127.0.0.1; };
//	allow-query     { localhost; };

#为了保证安全还需设置只允许从服务器进行区域传输
[root@centos8 ~]# grep "allow-transfer" /etc/named.conf 
	allow-transfer { 10.0.0.4;};
	
#在区域配置文件添加区域文件信息
[root@centos8 ~]# vim /etc/named.rfc1912.zones
zone "linux.org" {
   type master;
   file  "linux.org.zone";
};

#接着拷贝模板，编写区域文件信息
注意：要保证named账户对其有读权限，否则服务将无法启动
[root@centos8 ~]# cp -p /var/named/named.localhost /var/named/linux.org.zone
[root@centos8 ~]# cat /var/named/linux.org.zone 
$TTL 1D
@	IN SOA	master rname.invalid. (
					0	; serial
					1D	; refresh
					1H	; retry
					1W	; expire
					3H )	; minimum
	NS	master
	NS	slave
master	A	10.0.0.5
slave	A	10.0.0.4
www     A       10.0.0.3

#检查语法是否正确
[root@centos8 ~]# named-checkconf
[root@centos8 ~]# named-checkzone linux.org /var/named/linux.org.zone
zone linux.org/IN: loaded serial 0
OK

#启动服务
[root@centos8 ~]# systemctl start named

#如果服务已经启动，使用下面命令重新加载配置
[root@centos8 ~]# rndc reload 
```

#### 3.2.2 从DNS服务器配置

配置方式与主服务器大同小异，只需修改部分配置

安装服务端软件

```bash
[root@centos8 ~]# yum -y install bind 
```

编辑配置文件

```bash
[root@centos8 ~]# vim /etc/named.conf
//      listen-on port 53 { 127.0.0.1; };
//      allow-query     { localhost; };
		  allow-transfer { none;};

[root@centos8 ~]# vim /etc/named.rfc1912.zones
zone "linux.org" {
   type slave;
   masters {10.0.0.5;};   
         
   file "slaves/linux.org.slave";
};

#启动服务
[root@centos8 ~]# systemctl start named

#检查是否生成区域数据库文件
[root@centos8 ~]# ls /var/named/slaves/
linux.org.slave
```

#### 3.2.3 客户端测试主从DNS服务架构

```bash
#添加DNS服务器
[root@centos8 ~]# grep "DNS" /etc/sysconfig/network-scripts/ifcfg-ens160 
DNS1=10.0.0.5
DNS2=10.0.0.4

#重新加载网络服务
[root@centos8 ~]# nmcli con reload
[root@centos8 ~]# nmcli con up ens160

#查看是否生效
[root@centos8 ~]# cat /etc/resolv.conf 
# Generated by NetworkManager
nameserver 10.0.0.5
nameserver 10.0.0.4

#安装DNS客户端
[root@centos8 ~]# yum -y install bind-utils

#使用dig命令测试主从DNS服务器
[root@centos8 ~]# dig www.linux.org 

; <<>> DiG 9.11.20-RedHat-9.11.20-5.el8_3.1 <<>> www.linux.org
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 17919
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 3

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: f5289e5596176f597dfd6808609961ab4d370bc53826662a (good)
;; QUESTION SECTION:
;www.linux.org.			IN	A

;; ANSWER SECTION:
www.linux.org.		86400	IN	A	10.0.0.3

;; AUTHORITY SECTION:
linux.org.		86400	IN	NS	master.linux.org.
linux.org.		86400	IN	NS	slave.linux.org.

;; ADDITIONAL SECTION:
master.linux.org.	86400	IN	A	10.0.0.5
slave.linux.org.	86400	IN	A	10.0.0.4

;; Query time: 0 msec
;; SERVER: 10.0.0.5#53(10.0.0.5)
;; WHEN: Mon May 10 16:39:07 CST 2021
;; MSG SIZE  rcvd: 159

#ping命令测试
[root@centos8 ~]# ping www.linux.org
PING www.linux.org (10.0.0.3) 56(84) bytes of data.
64 bytes from 10.0.0.3 (10.0.0.3): icmp_seq=1 ttl=64 time=0.233 ms
64 bytes from 10.0.0.3 (10.0.0.3): icmp_seq=2 ttl=64 time=0.176 ms
64 bytes from 10.0.0.3 (10.0.0.3): icmp_seq=3 ttl=64 time=0.174 ms
```

## 4 智能DNS

### 4.1 GSLB和CDN

#### 4.1.1 GSLB

GSLB：Global Server Load Balance全局负载均衡

GSLB是对服务器和链路进行综合判断来决定由哪个地点的服务器来提供服务，实现异地服务器群服务 质量的保证

GSLB主要的目的是在整个网络范围内将用户的请求定向到最近的节点（或者区域）

GSLB分为基于DNS实现、基于重定向实现、基于路由协议实现，其中最通用的是基于DNS解析方式

#### 4.2.1 CDN

**内容分发网络**（英语：**C**ontent **D**elivery **N**etwork或**C**ontent **D**istribution **N**etwork，缩写：**CDN**）是指一种透过互联网互相连接的电脑网络系统，利用最靠近每位用户的服务器，更快、更可靠地将音乐、图片、视频、应用程序及其他文件发送给用户，来提供高性能、可扩展性及低成本的网络内容传递给用户。

CDN网络结合了GSLB与缓存技术

工作原理：

1. 用户向浏览器输入www.a.com这个域名，浏览器第一次发现本地没有dns缓存，则向网站的DNS 服务器请求
2. 网站的DNS域名解析器设置了CNAME，指向了www.a.tbcdn.com,请求指向了CDN网络中的智能 DNS负载均衡系统
3. 智能DNS负载均衡系统解析域名，把对用户响应速度最快的IP节点返回给用户；
4. 用户向该IP节点（CDN服务器）发出请求
5. 由于是第一次访问，CDN服务器会通过Cache内部专用DNS解析得到此域名的原web站点IP，向原 站点服务器发起请求，并在CDN服务器上缓存内容
6. 请求结果发给用户

4.2 智能DNS相关技术

### 4.2.1 bind中ACL

ACL：把一个或多个地址归并为一个集合，并通过一个统一的名称调用

注意：只能先定义后使用；因此一般定义在配置文件中，处于options的前面

范例：

```bash
acl beijingnet {
 172.16.0.0/16;
 10.10.10.10;
};
```

### 4.3 利用view实现智能DNS

#### 4.3.1 实验准备

实验前提

```bash
关闭SElinux
关闭防火墙
时间同步
```

环境要求：

```bash
需要五台主机
DNS主服务器和web服务器1：10.0.0.5/24，172.16.0.8/16
web服务器2：10.0.0.3/24
web服务器3：172.16.0.7/16
DNS客户端1：10.0.0.2/24
DNS客户端2：172.16.0.6/16
```

#### 4.3.2 实验步骤

1）DNS 服务器的网卡配置

```bash
#配置DNS服务器IP地址
[root@centos8 ~]# ip a
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:7f:44:8b brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.5/24 brd 10.0.0.255 scope global dynamic noprefixroute ens160
       valid_lft 1663sec preferred_lft 1663sec
4: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:0c:29:7f:44:95 brd ff:ff:ff:ff:ff:ff
    inet 172.16.0.8/16 brd 172.16.255.255 scope global noprefixroute ens192
       valid_lft forever preferred_lft forever
    inet6 fe80::20c:29ff:fe7f:4495/64 scope link tentative 
       valid_lft forever preferred_lft forever
```

2）主DNS服务端配置文件实现view

```bash
#安装服务端软件包
[root@centos8 ~]# yum -y install bind
#在最前面加入下面内容
acl chongqing {
        10.0.0.0/24
};
acl hunan {
        172.16.0.0/16
};
acl othernet {
        any
};
options {
//      listen-on port 53 { 127.0.0.1; };
//      allow-query     { localhost; };
};

#创建view来响应不同DNS查询请求
view chongqingview {
	match-clients {chongqing;};
	 zone "linux.org" {
        type master;
        file "linux.org.zone.cq";
      };

};
view hunanview {
        match-clients {hunan;};
        zone "linux.org" {
        type master;
        file "linux.org.zone.hn";
      };
	zone "." IN {
        type hint;
        file "named.ca";
      };

};
view otherview {
        match-clients {other;};
        zone "linux.org" {
        type master;
        file "linux.org.zone.other";
      };
	zone "." IN {
        type hint;
        file "named.ca";
      };
};

include "/etc/named.root.key";

#检查语法
[root@centos8 ~]# named-checkconf
```

3）创建区域数据库文件

```bash
[root@centos8 ~]# cat /var/named/linux.org.zone.cq 
$TTL 1D
@   IN SOA master admin.linux.org. (
                   0 ; serial
                   1D ; refresh
                   1H ; retry
                   1W ; expire
                   3H )   ; minimum
           NS   master
master     A   10.0.0.5
websrv     A   10.0.0.3                       
www       CNAME websrv

[root@centos8 ~]# cat /var/named/linux.org.zone.hn 
$TTL 1D
@   IN SOA master admin.linux.org. (
                   0 ; serial
                   1D ; refresh
                   1H ; retry
                   1W ; expire
                   3H )   ; minimum
           NS   master
master     A   10.0.0.5
websrv     A   172.16.0.7                       
www       CNAME websrv

[root@centos8 ~]# cat /var/named/linux.org.zone.other 
$TTL 1D
@   IN SOA master admin.linux.org. (
                   0 ; serial
                   1D ; refresh
                   1H ; retry
                   1W ; expire
                   3H )   ; minimum
           NS   master
master     A   10.0.0.5
websrv     A   127.0.0.1
www       CNAME websrv

#修改所属组
[root@centos8 ~]# chgrp named /var/named/linux.org.zone.cq 
[root@centos8 ~]# chgrp named /var/named/linux.org.zone.hn
[root@centos8 ~]# chgrp named /var/named/linux.org.zone.other

#启动服务
[root@centos8 ~]# systemctl start named
```

4）实现位于不同区域的三个WEB服务器

```bash
#在web服务器1：10.0.0.5/24实现http服务
[root@centos8 ~]# yum -y install httpd
[root@centos8 ~]# echo www.linux.org in Other > /var/www/html/index.html
[root@centos8 ~]# systemctl start httpd
[root@centos8 ~]# curl 10.0.0.5
www.linux.org in Other

#在web服务器2：10.0.0.3/24实现http服务
[root@centos8 yum.repos.d]# yum -y install httpd
[root@centos8 yum.repos.d]# echo www.linux.org in chongqing > /var/www/html/index.html
[root@centos8 yum.repos.d]# systemctl start httpd
[root@centos8 yum.repos.d]# curl 10.0.0.3
www.linux.org in chongqing

#在web服务器3：172.16.0.7/24实现http服务
[root@centos8 yum.repos.d]# yum -y install httpd
[root@centos8 yum.repos.d]# echo www.linux.org in chongqing > /var/www/html/index.html
[root@centos8 yum.repos.d]# systemctl start httpd
[root@centos8 yum.repos.d]# curl 172.16.0.7
www.linux.org in hunan
```

5） 客户端测试

注意：在测试前需确保DNS指向的是正确的服务器

```bash
#DNS客户端1：172.16.0.6/16
[root@centos8 ~]# hostname -I
172.16.0.6
[root@centos8 ~]# curl www.linux.org
www.linux.org in hunan

#DNS客户端2：10.0.0.2/24
[root@centos8 ~]# hostname -I
10.0.0.2 
[root@centos8 ~]# curl www.linux.org
www.linux.org in chongqing

#DNS客户端3：172.16.0.6/16
[root@centos8 ~]# curl www.linux.org
www.linux.org in Other
[root@centos8 ~]# hostname -I
10.0.0.5 172.16.0.8 
```

# 8.MySQL数据库

数据库：是以**一定方式**储存在一起、能予多个用户共享、具有尽可能小的冗余度、与应用程序彼此独立的数据集合。简单理解就是存储电子文件的仓库，用户可以对仓库中的资料进行增删改查等操作。

数据库管理系统（Database Management System）：简称DBMS,是为了管理数据库而设计的电脑软件系统，一般具有存储、截取、安全保障、备份等基础功能。

**数据库分类：**

关系型数据库（SQL）

- MySQL
- MariaDB
- Oracle

非关系型数据库（NOSQL）

- Redis
- MongoDB

键值数据库

- Dynamo
- LevelDB

对于数据库相关说明，例如：数据库的设计、数据库的模型、基本词汇解释MariaDB官方给出了相关知识库：https://mariadb.com/kb/en/database-theory/。

## 2 MySQL的安装和基本使用

### 2.1 MySQL介绍

#### 2.1.1 MySQL历史

MySQL原本是一个开源的关系数据库管理系统，不过后来被Sun公司收购，而后Sun公司又被（Oracle）公司收购，现在MySQL已成为Oracle旗下产品。

#### 2.1.2 MySQL分支

MySQL的三大主要分支：

- mysql
- mariadb
- percona server

其中mysql和mariadb是较为主流的开源数据库产品

他们的官方文档地址：

https://dev.mysql.com/doc/

https://mariadb.com/kb/en/

https://www.percona.com/software/mysql-database/percona-server

#### 2.1.3 MySQL的特性

- 插件式存储引擎：也称为“表类型”，存储管理器有多种实现版本，功能和特性可能均略有差别；用 户可根据需要灵活选择,Mysql5.5.5开始innoDB引擎是MYSQL默认引擎

   MyISAM ==> Aria

   InnoDB ==> XtraDB

- 单进程，多线程

- 诸多扩展和新特性

- 提供了较多测试组件

- 开源

### 2.2 MySQL安装

#### 2.2.1 安装方式介绍

- 源码包编译安装
- 二进制程序包（类似于windows中绿色安装）：解压至指定路径，简单配置即可使用
- 使用程序包管理器安装，也就是yum安装

#### 2.2.2 RPM包安装MySQL

这种方式较为简单，一般选择国内镜像仓库，配置yum源安装即可，与普通rpm包安装没有区别，缺点是不能定制配置，生产环境不建议采用这种方式安装

CentOS国内镜像：

https://mirrors.tuna.tsinghua.edu.cn/mariadb/yum/

https://mirrors.tuna.tsinghua.edu.cn/mysql/yum/

注意：这种方式安装完存在允许root用户空口令登陆，允许匿名用户登陆等安全问题，针对这个问题可以运行脚本`mysql_secure_installation`来设置

```mysql
设置数据库管理员root口令
禁止root远程登录
删除anonymous用户帐号
删除test数据库
```

### 2.3 MySQL组成

#### 2.3.1 客户端程序

- mysql：交互式的客户端工具

- mysqldump：备份工具，基于mysql协议向mysqld发起查询请求，并将查得的所有数据转换成 insert等写操作语句保存文本文件中

- mysqladmin：基于mysql协议管理mysqld

- mysqlimport：数据导入工具

  MyISAM存储引擎的管理工具：

- myisamchk：检查MyISAM库

- myisampack：打包MyISAM表，只读

#### 2.3.2 服务端程序

- mysqld_safe
- mysqld
- mysqld_multi 多实例 ，示例：mysqld_multi --example

#### 2.3.3 用户账号

mysql用户账号组成：

 'USERNAME'@'HOST‘

 HOST限制此用户可通过哪些远程主机连接mysql服务器

 支持使用通配符：

 % 匹配任意长度的任意字符 172.16.0.0/255.255.0.0 或 172.16.%.%

 _ 匹配任意单个字符

#### 2.3.4 mysql客户端命令

mysql命令格式：

```mysql
mysql [OPTIONS] [database]
```

常用选项：

- -u, --user= 用户名,默认为root
- -h, --host= 服务器主机,默认为localhost
- -p, --passowrd= 用户密码,建议使用-p,默认为空密码
- -P, --port= 服务器端口
- -S, --socket= 指定连接socket文件路径
- -C, --compress 启用压缩
- -e “SQL“ 执行SQL命令
- -V, --version 显示版本
- -v --verbose 显示详细信息
- --print-defaults 获取程序默认使用的配置

**mysql 使用模式：**

交互式

```mysql
客户端命令：本地执行，每个命令都完整形式和简写格式
mysql> \h, help
mysql> \u，use
mysql> \s，status
mysql> \!，system

服务端命令：通过mysql协议发往服务器执行并取回结果，命令末尾都必须使用命令结束符号，默认为分号
mysql>SELECT VERSION();
```

非交互式

```mysql
mysql –uUSERNAME -pPASSWORD < /path/somefile.sql
cat /path/somefile.sql | mysql –uUSERNAME -pPASSWORD
mysql>source   /path/from/somefile.sql
```

范例：

```mysql
#临时设置提示符的格式
mysql> \R (\u@\h) [\d]>\_
PROMPT set to '(\u@\h) [\d]>\_'
(root@localhost) [student]>
```

#### 2.3.5 mysqladmin命令

格式：

```mysql
mysqladmin [OPTIONS] command command....
```

范例：

```mysql
#查看mysql服务是否正常，如果正常提示mysqld is alive
mysqladmin -uroot -pcentos   ping

#关闭mysql服务，但mysqladmin命令无法开启
mysqladmin –uroot –pcentos shutdown

#创建数据库testdb
mysqladmin -uroot –pcentos   create testdb

#删除数据库testdb
mysqladmin -uroot -pcentos   drop testdb

#修改root密码
mysqladmin –uroot –pcentos password ‘magedu’

#日志滚动,生成新文件/var/lib/mysql/mariadb-bin.00000N
mysqladmin -uroot -pcentos flush-logs
```

#### 2.3.6 服务端配置

服务器端可以通过命令行或者配置文件进行配置，这里主要介绍配置文件配置方式。

服务器端配置文件：

```mysql
/etc/my.cnf #Global选项
/etc/mysql/my.cnf #Global选项
~/.my.cnf #User-specific 选项
```

配置文件格式：

```mysql
[mysqld]
[mysqld_safe]
[mysqld_multi]
[mysql]
[mysqldump]
[server]
[client]
格式：parameter = value
说明：_和- 相同
 1，ON，TRUE意义相同， 0，OFF，FALSE意义相同
```

服务器监听的两种socket地址：

- ip socket: 监听在tcp的3306端口，支持远程通信 ，侦听3306/tcp端口可以在绑定有一个或全部接 口IP上
- unix sock: 监听在sock文件上，仅支持本机通信, 如：/var/lib/mysql/mysql.sock) 说明：host为localhost 时自动使用unix sock

可以通过修改配置文件实现只侦听本地客户端

```mysql
vim /etc/my.cnf
[mysqld]
skip-networking=1
```

### 2.4 二进制安装MySQL

这里以安装MySQL5.7版本为例

官方安装手册：https://dev.mysql.com/doc/refman/5.7/en/binary-installation.html

MySQL下载地址：https://downloads.mysql.com/archives/community/

#### 2.4.1 创建MySQL用户和组

```mysql
[root@localhost local]# groupadd -r -g 306 mysql
[root@localhost local]# useradd -r -g 306 -u 306 -d /data/mysql -s /bin/false mysql
[root@localhost local]# getent passwd mysql
mysql:x:306:306::/data/mysql:/bin/false
```

#### 2.4.2 准备二进制文件

```mysql
[root@localhost local]# cd /usr/local
[root@localhost local]# tar xvf mysql-5.7.33-linux-glibc2.12-x86_64.tar.gz

[root@localhost local]# ln -s mysql-5.7.33-linux-glibc2.12-x86_64 mysql
[root@localhost local]# ls
bin  games    lib    libexec  mysql-5.7.33-linux-glibc2.12-x86_64         sbin   src
etc  include  lib64  mysql    mysql-5.7.33-linux-glibc2.12-x86_64.tar.gz  share
[root@localhost local]# chown -R root.root /usr/local/mysql/

#将路径添加至环境变量
[root@localhost mysql]# echo 'PATH=/app/boots-dev/bin/:$PATH' > /etc/profile.d/mysql.sh
[root@localhost mysql]# . /etc/profile.d/mysql.sh 
```

#### 2.4.3 准备配置文件

```mysql
[root@localhost local]# vim /etc/my.cnf
[mysqld]
datadir=/data/mysql
skip_name_resolve=1
socket=/data/mysql/mysql.sock        
log-error=/data/mysql/mysql.log
pid-file=/data/mysql/mysql.pid
[client]
socket=/data/mysql/mysql.sock
```

#### 2.4.4 创建数据库文件

```mysql
#初始命令会读取配置文件中的配置，由于之前已经指定了数据库文件位置，所以这里不用加入数据库路径选项
[root@localhost mysql]# /usr/local/mysql/bin/mysqld --initialize --user=mysql
2021-05-17T15:36:16.457156Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2021-05-17T15:36:18.652116Z 0 [Warning] InnoDB: New log files created, LSN=45790
2021-05-17T15:36:18.950575Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2021-05-17T15:36:19.034228Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 9ff6938c-b725-11eb-b707-000c2906d605.
2021-05-17T15:36:19.038515Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2021-05-17T15:36:19.564502Z 0 [Warning] CA certificate ca.pem is self signed.
2021-05-17T15:36:19.707592Z 1 [Note] A temporary password is generated for root@localhost: !Mn4hek<bO%.		#系统生成的首次登陆密码

#查看数据库文件是否生成
[root@localhost mysql]# ls /data/mysql/
auto.cnf    client-cert.pem  ibdata1      mysql               public_key.pem   sys
ca-key.pem  client-key.pem   ib_logfile0  performance_schema  server-cert.pem
ca.pem      ib_buffer_pool   ib_logfile1  private_key.pem     server-key.pem
```

#### 2.4.5 准备服务脚本并启动

```mysql
#准备服务脚本
[root@localhost mysql]# cp support-files/mysql.server /etc/init.d/mysqld
[root@localhost mysql]# chkconfig --add mysqld
[root@localhost mysql]# chkconfig --list

Note: This output shows SysV services only and does not include native
      systemd services. SysV configuration data might be overridden by native
      systemd configuration.

      If you want to list systemd services use 'systemctl list-unit-files'.
      To see services enabled on particular target use
      'systemctl list-dependencies [target]'.

mysqld         	0:off	1:off	2:on	3:on	4:on	5:on	6:off
netconsole     	0:off	1:off	2:off	3:off	4:off	5:off	6:off
network        	0:off	1:off	2:on	3:on	4:on	5:on	6:off

#启动数据库
[root@localhost mysql]# service mysqld start
Starting MySQL.Logging to '/data/mysql/localhost.localdomain.err'.
... SUCCESS! 

#查看是否启动成功
[root@localhost mysql]# ss -ntl
State       Recv-Q Send-Q        Local Address:Port                       Peer Address:Port         
LISTEN      0      128                       *:22                                    *:*       
LISTEN      0      80                     [::]:3306                               [::]:*   
```

#### 2.4.6 修改默认口令

```mysql
[root@localhost mysql]# mysqladmin -uroot -p'aY4lhQ/uyB+%' password database
mysqladmin: [Warning] Using a password on the command line interface can be insecure.
Warning: Since password will be sent to server in plain text, use ssl connection to ensure password safety.

#测试是否能登陆
[root@localhost mysql]# mysql -uroot -pdatabase
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
......

#安全初始化
[root@localhost mysql]# mysql_secure_installation

Securing the MySQL server deployment.

Enter password for user root: 
......
```

### 2.5 源码编译安装

这里以安装MySQL5.7版本为例

下载源码包：https://dev.mysql.com/downloads/mysql/

官方参考文档：https://dev.mysql.com/doc/refman/5.7/en/installing-source-distribution.html

注意：安装过程应保证4G以上内存

#### 2.5.1 安装相关依赖包

```mysql
[root@localhost ~]# yum -y install bison bison-devel zlib-devel libcurl-devel libarchive-devel
boost-devel  gcc gcc-c++ cmake ncurses-devel gnutls-devel libxml2-devel
openssl-devel libevent-devel libaio-devel
```

#### 2.5.2 创建用户

```mysql
[root@localhost local]# useradd -r -s /sbin/nologin -d /data/mysql mysql
```

#### 2.5.3 源码编译安装

```mysql
[root@localhost local]# ls
bin  etc  games  include  lib  lib64  libexec  mysql-5.7.34.tar.gz  sbin  share  src

#解压缩源码包
[root@localhost local]# tar xvf mysql-5.7.34.tar.gz

#进入目录编译安装
[root@localhost local]# cd mysql-5.7.34
[root@localhost mysql-5.7.34]# mkdir bld
[root@localhost mysql-5.7.34]# cd bld

#注意：5.7版本mysql要求Boost版本为1.59.0，由于yum版本不满足要求，所以还需再官网下载boost1.59.0版本压缩包，这里使用-DWITH_BOOST可以指定boost1.59.0版本压缩包的路径，即boost_1_59_0.tar.gz的位置
[root@localhost bld]# cmake .. -DCMAKE_INSTALL_PREFIX=/app/mysql \
-DMYSQL_DATADIR=/data/mysql/ \
-DSYSCONFDIR=/etc/ \
-DMYSQL_USER=mysql \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITHOUT_MROONGA_STORAGE_ENGINE=1 \
-DWITH_DEBUG=0 \
-DWITH_READLINE=1 \
-DWITH_SSL=system \
-DWITH_ZLIB=system \
-DWITH_BOOST=/app/boots \	
-DWITH_LIBWRAP=0 \
-DENABLED_LOCAL_INFILE=1 \
-DMYSQL_UNIX_ADDR=/data/mysql/mysql.sock \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci
[root@localhost bld]# make -j 16
#编译过程由于内存不足导致报错c++: internal compiler error，添加内存重新编译即可
[root@localhost bld]# make install 
```

#### 2.5.4 配置数据库

```mysql
#配置环境变量
[root@localhost bld]# echo 'PATH=/app/mysql/bin:$PATH' > /etc/profile.d/mysql.sh
[root@localhost bld]# . /etc/profile.d/mysql.sh

#生成数据库文件
[root@localhost mysql]# mysqld --initialize --datadir=/data/mysql/ --user=mysql
2021-05-18T17:55:55.406121Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2021-05-18T17:55:55.663535Z 0 [Warning] InnoDB: New log files created, LSN=45790
2021-05-18T17:55:55.703042Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2021-05-18T17:55:55.760620Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 4b4b5646-b802-11eb-8afd-000c2906d605.
2021-05-18T17:55:55.772427Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2021-05-18T17:55:56.274206Z 0 [Warning] CA certificate ca.pem is self signed.
2021-05-18T17:55:56.660120Z 1 [Note] A temporary password is generated for root@localhost: na6t_YuJgrAs

#配置数据库配置文件
[root@localhost mysql]# vim /etc/my.cnf
[mysqld]
datadir=/data/mysql
skip_name_resolve=1
socket=/data/mysql/mysql.sock        
log-error=/data/mysql/mysql.log
pid-file=/data/mysql/mysql.pid
[client]
socket=/data/mysql/mysql.sock

#拷贝服务启动脚本
[root@localhost mysql]# cp support-files/mysql.server /etc/init.d/mysqld
[root@localhost mysql]# chkconfig --add mysqld
[root@localhost mysql]# service mysqld start
Starting MySQL.Logging to '/data/mysql/mysql.log'.
 SUCCESS! 
```

#### 2.5.5 安全初始化

```mysql
[root@localhost mysql]# mysql_secure_installation

#测试是否能登陆
[root@localhost mysql]# mysql -uroot -pdatabase
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 7
Server version: 5.7.34 Source distribution
```

## 3 SQL语言

### 3.1 关系型数据库的常见组件

- 数据库：database
- 表：table，行：row 列：column
- 索引：index
- 视图：view
- 用户：user
- 权限：privilege
- 存储过程：procedure
- 存储函数：function
- 触发器：trigger
- 事件调度器：event scheduler，任务计划

### 3.2 SQL语言的语法标准

#### 3.2.1 SQL语法

SQL语句不区分大小写，建议使用大写

SQL语句可以多行或者单行书写，通常以;结尾

关键词不能跨多行或简写

注释：

- SQL标准：

  -- 注释内容 用于单行注释，注意有空格

  /*注释内容*/ 多行注释

- MySQL注释：

  \# 注释内容

#### 3.2.2 数据库对象和命名

数据库的组件（对象）：

数据库、表、索引、视图、用户、存储过程、函数、触发器、事件调度器等

命名规则：

- 必须以字母开头，可包括数字和三个特殊字符（# _ $）
- 不要使用MySQL的保留字
- 同一database(Schema)下的对象不能同名

#### 3.2.3 SQL语句分类

- DDL: Data Defination Language 数据定义语言

  CREATE，DROP，ALTER

- DML: Data Manipulation Language 数据操纵语言 INSERT，DELETE，UPDATE

- DQL：Data Query Language 数据查询语言

  SELECT

- DCL：Data Control Language 数据控制语言

  GRANT，REVOKE，COMMIT，ROLLBACK

官方帮助：https://dev.mysql.com/doc/refman/8.0/en/sql-statements.html

本地mysql提供的帮助

```mysql
mysql> HELP KEYWORD
```

### 3.3 数据类型

官方数据类型参考链接：https://dev.mysql.com/doc/refman/8.0/en/data-types.html

对于整数数据类型，*`M`*指示最大显示宽度。对于浮点和定点数据类型， *`M`*是可以存储的总位数。

#### 3.3.1 整数型

tinyint(m) 1个字节 范围(-128~127)

smallint(m) 2个字节 范围(-32768~32767)

mediumint(m) 3个字节 范围(-8388608~8388607)

int(m) 4个字节 范围(-2147483648~2147483647)

bigint(m) 8个字节 范围(+-9.22*10的18次方)

#### 3.3.2 浮点型

float(m,d) 单精度浮点型 8位精度(4字节) m总个数，d小数位 double(m,d) 双精度浮点型16位精度(8字节) m总个数，d小数位 设一个字段定义为float(6,3)，如果插入一个数123.45678,实际数据库里存的是123.457，但总个数还以 实际为准，即6位

#### 3.3.3 定点数

在数据库中存放的是精确值,存为十进制

decimal(m,d) 参数m<65 是总个数，d<30且 d<m 是小数位

MySQL5.0和更高版本将数字打包保存到一个二进制字符串中（每4个字节存9个数字）。

#### 3.3.4 字符串

char(n) 固定长度，最多255个字符

varchar(n) 可变长度，最多65535个字符

tinytext 可变长度，最多255个字符

text 可变长度，最多65535个字符

mediumtext 可变长度，最多2的24次方-1个字符

longtext 可变长度，最多2的32次方-1个字符

BINARY(M) 固定长度，可存二进制或字符，长度为0-M字节 VARBINARY(M) 可变长度，可存二进制或字符，允许长度为0-M字节

内建类型：ENUM枚举, SET集合

#### 3.3.5 二进制数据BLOB

BLOB和text存储方式不同，TEXT以文本方式存储，英文存储区分大小写，而Blob以二进制方式存储， 不分大小写

#### 3.3.6 日期时间类型

date 日期 '2008-12-2'

time 时间 '12:25:36'

datetime 日期时间 '2008-12-2 22:06:44'

timestamp 自动存储记录修改时间

YEAR(2), YEAR(4)：年份

timestamp字段里的时间数据会随其他字段修改的时候自动刷新，这个数据类型的字段可以存放这条记 录最后被修改的时间

#### 3.3.7 修饰符

NULL 数据列可包含NULL值，默认值

NOT NULL 数据列不允许包含NULL值，*为必填选项

DEFAULT 默认值

PRIMARY KEY 主键，所有记录中此字段的值不能重复，且不能为NULL

UNIQUE KEY 唯一键，所有记录中此字段的值不能重复，但可以为NULL

CHARACTER SET name 指定一个字符集

AUTO_INCREMENT 自动递增，适用于整数类型

UNSIGNED 无符号

### 3.4 DDL语句

#### 3.4.1 create table

获取帮助：

```mysql
HELP CREATE TABLE
```

官方文档：https://dev.mysql.com/doc/refman/8.0/en/create-table.html

创建表的三种方法

(1) 直接创建新表

格式：

```mysql
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
    (create_definition,...)
    [table_options]
    [partition_options]
```

范例：

```mysql
#创建一个名为student的表，使其结构如下
(root@localhost) [test]> desc student;
+--------+---------------------+------+-----+---------+----------------+
| Field  | Type                | Null | Key | Default | Extra          |
+--------+---------------------+------+-----+---------+----------------+
| id     | int(10) unsigned    | NO   | PRI | NULL    | auto_increment |
| name   | varchar(20)         | NO   |     | NULL    |                |
| age    | tinyint(3) unsigned | YES  |     | NULL    |                |
| gender | enum('M','F')       | YES  |     | M       |                |
+--------+---------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)

#语句
(root@localhost) [test]>create table student (
id int unsigned auto_increment primary key,	#无符号的int型整数，并且自动增加设置为主键
name varchar(20) not null,	#可变字符不为空
age tinyint unsigned,	#无符号tinyint类型数据
gender enum('M','F') default 'M'	#将性别设置为一个enum枚举字符串对象，值为M和F
)ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8;	#使用innodb引擎，步进为10，默认字符集为utf8
```

（2)通过查询现表创建新表

格式：

```mysql
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
    [(create_definition,...)]
    [table_options]
    [partition_options]
    [IGNORE | REPLACE]
    [AS] query_expression
```

(3) 通过复制现存的表的表结构创建，但不复制数据

格式：

```mysql
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] tbl_name
    { LIKE old_tbl_name | (LIKE old_tbl_name) }
```

#### 3.4.2 查看表

查看支持的engine类型

```mysql
SHOW ENGINES;
```

查看表：

```mysql
SHOW TABLES [FROM db_name]
```

查看表结构：

```mysql
DESC [db_name.]tb_name
SHOW COLUMNS FROM [db_name.]tb_name
```

查看表创建命令：

```mysql
SHOW CREATE TABLE tbl_name
```

查看表状态：

```mysql
SHOW TABLE STATUS LIKE 'tbl_name’
```

查看库中所有表状态：

```mysql
SHOW TABLE STATUS FROM db_name
```

#### 3.4.3 修改和删除表

删除表

```mysql
DROP [TEMPORARY] TABLE [IF EXISTS]
    tbl_name [, tbl_name] ...
    [RESTRICT | CASCADE]
```

修改表

```mysql
ALTER TABLE tbl_name
    [alter_option [, alter_option] ...]
    [partition_options]
```

官方文档：https://dev.mysql.com/doc/refman/8.0/en/alter-table.html

范例：

```mysql
ALTER TABLE students RENAME s1;
ALTER TABLE s1 ADD phone varchar(11) AFTER name;
ALTER TABLE s1 MODIFY phone int;
ALTER TABLE s1 CHANGE COLUMN phone mobile char(11);
ALTER TABLE s1 DROP COLUMN mobile;
ALTER TABLE s1 character set utf8;
ALTER TABLE s1 change name name varchar(20) character set utf8;
ALTER TABLE students ADD gender ENUM('m','f');
ALETR TABLE students CHANGE id sid int UNSIGNED NOT NULL PRIMARY KEY;
ALTER TABLE students DROP age;
DESC students;
```

### 3.5 DML语句

#### 3.5.1 insert

一次插入一行或多行数据

格式：

```mysql
INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    [(col_name [, col_name] ...)]
    {VALUES | VALUE} (value_list) [, (value_list)] ...
    [ON DUPLICATE KEY UPDATE assignment_list]

INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    SET assignment_list
    [ON DUPLICATE KEY UPDATE assignment_list]

INSERT [LOW_PRIORITY | HIGH_PRIORITY] [IGNORE]
    [INTO] tbl_name
    [PARTITION (partition_name [, partition_name] ...)]
    [(col_name [, col_name] ...)]
    SELECT ...
    [ON DUPLICATE KEY UPDATE assignment_list]
```

也可以写成

```mysql
INSERT tbl_name [(col1,...)] VALUES (val1,...), (val21,...)
```

#### 3.5.2 update

格式：

```mysql
UPDATE [LOW_PRIORITY] [IGNORE] table_reference
    SET assignment_list
    [WHERE where_condition]
    [ORDER BY ...]
    [LIMIT row_count]
```

**注意：一定要加入条件限制，否则会更新所有符合条件数据，使用以下方式来避免这类错误**

```mysql
mysql -U | --safe-updates| --i-am-a-dummy
[root@centos8 ~]#vim /etc/my.cnf
[mysql]
safe-updates
```

#### 3.5.3 delete

格式：

```mysql
DELETE [LOW_PRIORITY] [QUICK] [IGNORE] FROM tbl_name
   [WHERE where_condition]
   [ORDER BY ...]
   [LIMIT row_count]
```

**注意：删除表中数据，但不会自动缩减数据文件的大小。**

如果想清空表，保留表结构，也可以使用下面语句，此语句会自动缩减数据文件的大小。

```mysql
TRUNCATE TABLE tbl_name;
```

缩减表大小

```mysql
OPTIMIZE TABLE tb_name
```

### 3.6 DQL语句

#### 3.6.1 单表查询

格式：

```mysql
SELECT
    [ALL | DISTINCT | DISTINCTROW ]
    [HIGH_PRIORITY]
    [STRAIGHT_JOIN]
    [SQL_SMALL_RESULT] [SQL_BIG_RESULT] [SQL_BUFFER_RESULT]
    [SQL_NO_CACHE] [SQL_CALC_FOUND_ROWS]
    select_expr [, select_expr] ...
    [into_option]
    [FROM table_references
      [PARTITION partition_list]]
    [WHERE where_condition]
    [GROUP BY {col_name | expr | position}, ... [WITH ROLLUP]]
    [HAVING where_condition]
    [WINDOW window_name AS (window_spec)
        [, window_name AS (window_spec)] ...]
    [ORDER BY {col_name | expr | position}
      [ASC | DESC], ... [WITH ROLLUP]]
    [LIMIT {[offset,] row_count | row_count OFFSET offset}]
    [into_option]
    [FOR {UPDATE | SHARE}
        [OF tbl_name [, tbl_name] ...]
        [NOWAIT | SKIP LOCKED]
      | LOCK IN SHARE MODE]
    [into_option]
```

**7⼤⼦句书写顺序**

（1）from：从哪些表中筛选

（2）on：关联多表查询时，去除笛卡尔积

（3）where：从表中筛选的条件

 过滤条件：布尔型表达式

 算术操作符：+, -, *, /, %

 比较操作符：=,<=>（相等或都为空）, <>, !=(非标准SQL), >, >=, <, <=

 BETWEEN min_num AND max_num

 IN (element1, element2, ...)

 IS NULL

 IS NOT NULL

 DISTINCT 去除重复行，范例：SELECT DISTINCT gender FROM students;

 LIKE: % 任意长度的任意字符 _ 任意单个字符

 RLIKE：正则表达式，索引失效，不建议使用

 REGEXP：匹配字符串可用正则表达式书写模式，同上

 逻辑操作符：NOT，AND，OR，XOR

（4）group by：根据指定的条件把查询结果进行“分组”以用于做“聚合”运算

 常见聚合函数：avg(), max(), min(), count(), sum()

 一旦分组group by ，select语句后只跟分组的字段，聚合函数

（5）having：在统计结果中再次筛选

（6）order by：排序

 降序：desc

 升序：要么默认，要么加asc

（7）limit：对查询的结果进行输出行数数量限制

必须按照（1）-（7）的顺序【编写】⼦句。

select语句执行流程

```mysql
FROM Clause --> WHERE Clause --> GROUP BY --> HAVING Clause -->SELECT --> ORDER
BY --> LIMIT
```

having与where的区别？

（1）where是从表中筛选的条件，而having是分组（统计）结果中再次筛选

（2）where后⾯不能加“分组/聚合函数”，而having后⾯可以跟分组函数

范例：

```mysql
#查询每个部门的男生的人数，并且显示人数超过5人的，按照人数降序排列，
(root@localhost) [test]>  SELECT did,COUNT(*) "人数"
    -> FROM t_employee
    -> WHERE gender = '男'
    -> GROUP BY did
    -> HAVING COUNT(*)>5
    -> ORDER BY 人数 DESC;
+------+--------+
| did  | 人数   |
+------+--------+
|    1 |     11 |
+------+--------+
1 row in set (0.00 sec)

#查询各个部门的岗位工资总和是多少
mysql> SELECT department_id,job_id,SUM(salary)
    -> FROM employees
    -> GROUP BY department_id,job_id;
+---------------+------------+-------------+
| department_id | job_id     | SUM(salary) |
+---------------+------------+-------------+
|            90 | AD_PRES    |    24000.00 |
|            90 | AD_VP      |    34000.00 |
|            60 | IT_PROG    |    28800.00 |
|           100 | FI_MGR     |    12000.00 |
|           100 | FI_ACCOUNT |    39600.00 |
|            30 | PU_MAN     |    11000.00 |
|            30 | PU_CLERK   |    13900.00 |

#获取最小⼯资小于2000的职位
mysql> select job_id,min(salary)
    -> from employees
    -> group by job_id
    -> having min(salary)<3000
    -> order by min(salary);
+----------+-------------+
| job_id   | min(salary) |
+----------+-------------+
| ST_CLERK |     2100.00 |
| PU_CLERK |     2500.00 |
| SH_CLERK |     2500.00 |
+----------+-------------+
```

#### 3.6.2 多表查询

##### 3.6.2.1 子查询

在SQL语句嵌套着查询语句，不过性能较差

**where型**

```mysql
#返回单个值
(root@localhost) [hellodb]> select name,age from students where age > (select avg(age) from teachers);
+-------------+-----+
| name        | age |
+-------------+-----+
| Sun Dasheng | 100 |
+-------------+-----+
1 row in set (0.01 sec)

#使用in返回多个值
(root@localhost) [hellodb]> select name,age from students where age in (select age from teachers);
+-------------+-----+
| name        | age |
+-------------+-----+
| Shi Zhongyu |  77 |
+-------------+-----+
1 row in set (0.00 sec)
```

**exists型**

```mysql
#选出ID和老师相同的学生
(root@localhost) [hellodb]> select * from students s where exists (select * from teachers t where s.teacherid=t.tid);
+-------+-------------+-----+--------+---------+-----------+
| StuID | Name        | Age | Gender | ClassID | TeacherID |
+-------+-------------+-----+--------+---------+-----------+
|     1 | Shi Zhongyu |  77 | M      |       2 |         3 |
|     4 | Ding Dian   |  32 | M      |       4 |         4 |
|     5 | Yu Yutong   |  26 | M      |       3 |         1 |
+-------+-------------+-----+--------+---------+-----------+
3 rows in set (0.00 sec)
上述语句把students的记录逐条代入到Exists后面的子查询中，如果子查询结果集不为空，即说
明存在，那么这条students的记录出现在最终结果集，否则被排除
```

**from型**

```mysql
(root@localhost) [hellodb]> SELECT s.aage,s.ClassID FROM (SELECT avg(Age) AS aage,ClassID FROM students
    -> WHERE ClassID IS NOT NULL GROUP BY ClassID) AS s WHERE s.aage>30;
+---------+---------+
| aage    | ClassID |
+---------+---------+
| 54.3333 |       2 |
| 46.0000 |       5 |
+---------+---------+
2 rows in set (0.01 sec)
```

##### 3.6.2.2 联合查询

union实现多个表之间纵向合并

范例：

```mysql
#将teachers和students表纵向合并
(root@localhost) [hellodb]> SELECT Name,Age FROM students UNION SELECT Name,Age FROM teachers;
+---------------+-----+
| Name          | Age |
+---------------+-----+
| Shi Zhongyu   |  77 |
| Shi Potian    |  22 |
| Xie Yanke     |  53 |
| Ding Dian     |  32 |
| Yu Yutong     |  26 |
| Shi Qing      |  46 |
| Xi Ren        |  19 |
......
```

##### 3.6.2.3 交叉连接(笛卡尔乘积)

利用 cross join实现

```mysql
(root@localhost) [hellodb]>  select * from teachers , students;
(root@localhost) [hellodb]> select * from students cross join teachers;
+-----+---------------+-----+--------+-------+---------------+-----+--------+---------+-----------+
| TID | Name          | Age | Gender | StuID | Name          | Age | Gender | ClassID | TeacherID |
+-----+---------------+-----+--------+-------+---------------+-----+--------+---------+-----------+
|   1 | Song Jiang    |  45 | M      |     1 | Shi Zhongyu   |  77 | M      |       2 |         3 |
|   2 | Zhang Sanfeng |  94 | M      |     1 | Shi Zhongyu   |  77 | M      |       2 |         3 |
|   3 | Miejue Shitai |  77 | F      |     1 | Shi Zhongyu   |  77 | M      |       2 |         3 |
|   4 | Lin Chaoying  |  93 | F      |     1 | Shi Zhongyu   |  77 | M      |       2 |         3 |
|   1 | Song Jiang    |  45 | M      |     2 | Shi Potian    |  22 | M      |       1 |         7 |
```

##### 3.6.2.4 关联查询

[![image](https://img2020.cnblogs.com/blog/2222624/202106/2222624-20210628221803206-532958897.png)](https://img2020.cnblogs.com/blog/2222624/202106/2222624-20210628221803206-532958897.png)

**内连接**

```mysql
#格式
select 字段列表
from A表 inner join B表
on 关联条件
where 等其他子句;

或

select 字段列表
from A表 , B表
where 关联条件 and 等其他子句;

#查询学生的老师
(root@localhost) [hellodb]> select * from students inner join teachers on students.teacherid=teachers.tid;
+-------+-------------+-----+--------+---------+-----------+-----+---------------+-----+--------+
| StuID | Name        | Age | Gender | ClassID | TeacherID | TID | Name          | Age | Gender |
+-------+-------------+-----+--------+---------+-----------+-----+---------------+-----+--------+
|     1 | Shi Zhongyu |  77 | M      |       2 |         3 |   3 | Miejue Shitai |  77 | F      |
|     4 | Ding Dian   |  32 | M      |       4 |         4 |   4 | Lin Chaoying  |  93 | F      |
|     5 | Yu Yutong   |  26 | M      |       3 |         1 |   1 | Song Jiang    |  45 | M      |
+-------+-------------+-----+--------+---------+-----------+-----+---------------+-----+--------+
3 rows in set (0.00 sec)

(root@localhost) [hellodb]> select * from students,teachers where students.teacherid=teachers.tid;
+-------+-------------+-----+--------+---------+-----------+-----+---------------+-----+--------+
| StuID | Name        | Age | Gender | ClassID | TeacherID | TID | Name          | Age | Gender |
+-------+-------------+-----+--------+---------+-----------+-----+---------------+-----+--------+
|     1 | Shi Zhongyu |  77 | M      |       2 |         3 |   3 | Miejue Shitai |  77 | F      |
|     4 | Ding Dian   |  32 | M      |       4 |         4 |   4 | Lin Chaoying  |  93 | F      |
|     5 | Yu Yutong   |  26 | M      |       3 |         1 |   1 | Song Jiang    |  45 | M      |
+-------+-------------+-----+--------+---------+-----------+-----+---------------+-----+--------+
3 rows in set (0.00 sec)
```

**左右外连接**

```mysql
#左外连接格式
select 字段列表
from A表 left join B表
on 关联条件
where 等其他子句;

#如果连接的部分右表没有值则会显示为null
(root@localhost) [hellodb]>  select s.stuid,s.name,s.age,s.teacherid,t.tid,t.name,t.age
    -> from students as s left outer join teachers as t on s.teacherid=t.tid;
+-------+---------------+-----+-----------+------+---------------+------+
| stuid | name          | age | teacherid | tid  | name          | age  |
+-------+---------------+-----+-----------+------+---------------+------+
|     5 | Yu Yutong     |  26 |         1 |    1 | Song Jiang    |   45 |
|     1 | Shi Zhongyu   |  77 |         3 |    3 | Miejue Shitai |   77 |
|     4 | Ding Dian     |  32 |         4 |    4 | Lin Chaoying  |   93 |
|     2 | Shi Potian    |  22 |         7 | NULL | NULL          | NULL |
|     3 | Xie Yanke     |  53 |        16 | NULL | NULL          | NULL |
|     6 | Shi Qing      |  46 |      NULL | NULL | NULL          | NULL |
|     7 | Xi Ren        |  19 |      NULL | NULL | NULL          | NULL |
|     8 | Lin Daiyu     |  17 |      NULL | NULL | NULL          | NULL |
|     9 | Ren Yingying  |  20 |      NULL | NULL | NULL          | NULL |
|    10 | Yue Lingshan  |  19 |      NULL | NULL | NULL          | NULL |
|    11 | Yuan Chengzhi |  23 |      NULL | NULL | NULL          | NULL |
|    12 | Wen Qingqing  |  19 |      NULL | NULL | NULL          | NULL |
|    13 | Tian Boguang  |  33 |      NULL | NULL | NULL          | NULL |
|    14 | Lu Wushuang   |  17 |      NULL | NULL | NULL          | NULL |
|    15 | Duan Yu       |  19 |      NULL | NULL | NULL          | NULL |
|    16 | Xu Zhu        |  21 |      NULL | NULL | NULL          | NULL |
|    17 | Lin Chong     |  25 |      NULL | NULL | NULL          | NULL |
|    18 | Hua Rong      |  23 |      NULL | NULL | NULL          | NULL |
|    19 | Xue Baochai   |  18 |      NULL | NULL | NULL          | NULL |
|    20 | Diao Chan     |  19 |      NULL | NULL | NULL          | NULL |
|    21 | Huang Yueying |  22 |      NULL | NULL | NULL          | NULL |
|    22 | Xiao Qiao     |  20 |      NULL | NULL | NULL          | NULL |
|    23 | Ma Chao       |  23 |      NULL | NULL | NULL          | NULL |
|    24 | Xu Xian       |  27 |      NULL | NULL | NULL          | NULL |
|    25 | Sun Dasheng   | 100 |      NULL | NULL | NULL          | NULL |
+-------+---------------+-----+-----------+------+---------------+------+
25 rows in set (0.01 sec)

#右外连接格式
select 字段列表
from A表 right join B表
on 关联条件
where 等其他子句;

(root@localhost) [hellodb]> select * from students s right outer join teachers t on s.teacherid=t.tid;
+-------+-------------+------+--------+---------+-----------+-----+---------------+-----+--------+
| StuID | Name        | Age  | Gender | ClassID | TeacherID | TID | Name          | Age | Gender |
+-------+-------------+------+--------+---------+-----------+-----+---------------+-----+--------+
|     1 | Shi Zhongyu |   77 | M      |       2 |         3 |   3 | Miejue Shitai |  77 | F      |
|     4 | Ding Dian   |   32 | M      |       4 |         4 |   4 | Lin Chaoying  |  93 | F      |
|     5 | Yu Yutong   |   26 | M      |       3 |         1 |   1 | Song Jiang    |  45 | M      |
|  NULL | NULL        | NULL | NULL   |    NULL |      NULL |   2 | Zhang Sanfeng |  94 | M      |
+-------+-------------+------+--------+---------+-----------+-----+---------------+-----+--------+
```

##### 3.6.2.5 完全外连接

Mysql不支持上图语句，所以这里我们使用左外的A,union,右外的B

```mysql
#格式
select 字段列表
from A表 left join B表
on 关联条件
where 等其他子句

union

select 字段列表
from A表 right join B表
on 关联条件
where 等其他子句;

(root@localhost) [hellodb]> select * from students left join teachers  on
    -> students.teacherid=teachers.tid
    -> union
    -> select * from students right join teachers on students.teacherid=teachers.tid;
 所得出的表就是左外和右外表相联合
```

##### 3.6.2.6 自连接

两个关联查询的表是同⼀张表，通过取别名的⽅式来虚拟成两张表

```mysql
#格式
select 字段列表
from 表名 别名1 inner/left/right join 表名 别名2
on 别名1.关联字段 = 别名2的关联字段
where 其他条件

#查询员工的编号，姓名，薪资和他领导的编号，姓名，薪资
#这些数据全部在员工表中
#把t_employee表，即当做员工表，又当做领导表
mysql> SELECT emp.eid "员工编号",emp.ename "员工姓名",emp.salary "工资",mgr.eid "领导编号",mgr.ename "领导姓名",mgr.salary "领 导工资"
    -> FROM t_employee emp INNER JOIN t_employee mgr
    -> ON emp.mid=mgr.eid;
+--------------+--------------+---------+--------------+--------------+--------------+
| 员工编号     | 员工姓名     | 工资    | 领导编号     | 领导姓名     | 领导工资     |
+--------------+--------------+---------+--------------+--------------+--------------+
|            1 | 孙红雷       | 8000.46 |            7 | 贾乃亮       |        15700 |
|            2 | 何炅         | 7000.67 |            7 | 贾乃亮       |        15700 |
|            3 | 邓超         |    8000 |            7 | 贾乃亮       |        15700 |
|            4 | 黄晓明       |    9456 |           22 | 刘烨         |       130990 |
|            5 | 陈赫         |    8567 |            7 | 贾乃亮       |        15700 |
|            6 | 韩庚         |   12000 |            7 | 贾乃亮       |        15700 |
```

练习：

1、 导入hellodb.sql生成数据库

(1) 在students表中，查询年龄大于25岁，且为男性的同学的名字和年龄

```mysql
(root@localhost) [hellodb]> select Name,age from students where age > 25 and Gender='M';
+--------------+-----+
| Name         | age |
+--------------+-----+
| Shi Zhongyu  |  77 |
| Xie Yanke    |  53 |
| Ding Dian    |  32 |
| Yu Yutong    |  26 |
| Shi Qing     |  46 |
| Tian Boguang |  33 |
| Xu Xian      |  27 |
| Sun Dasheng  | 100 |
+--------------+-----+
8 rows in set (0.00 sec)
```

(2) 以ClassID为分组依据，显示每组的平均年龄

```mysql
(root@localhost) [hellodb]> select ClassID,avg(age) from students where ClassID is not null group by ClassID
D;
+---------+----------+
| ClassID | avg(age) |
+---------+----------+
|       1 |  20.5000 |
|       2 |  54.3333 |
|       3 |  20.2500 |
|       4 |  24.7500 |
|       5 |  46.0000 |
|       6 |  20.7500 |
|       7 |  19.6667 |
+---------+----------+
7 rows in set (0.00 sec)
```

(3) 显示第2题中平均年龄大于30的分组及平均年龄

```mysql
(root@localhost) [hellodb]> select ClassID,avg(age) from students where ClassID is not null group by ClassID having avg(age) > 30;
+---------+----------+
| ClassID | avg(age) |
+---------+----------+
|       2 |  54.3333 |
|       5 |  46.0000 |
+---------+----------+
2 rows in set (0.01 sec)
```

(4) 显示以L开头的名字的同学的信息

```mysql
(root@localhost) [hellodb]> select * from students where name like 'L%';
+-------+-------------+-----+--------+---------+-----------+
| StuID | Name        | Age | Gender | ClassID | TeacherID |
+-------+-------------+-----+--------+---------+-----------+
|     8 | Lin Daiyu   |  17 | F      |       7 |      NULL |
|    14 | Lu Wushuang |  17 | F      |       3 |      NULL |
|    17 | Lin Chong   |  25 | M      |       4 |      NULL |
+-------+-------------+-----+--------+---------+-----------+
3 rows in set (0.01 sec)
```

2、数据库授权magedu用户，允许192.168.1.0/24网段可以连接mysql

```mysql
(root@localhost) [hellodb]> GRANT ALL ON mysql TO magedu@'192.168.1.%' IDENTIFIED BY 'database';
Query OK, 0 rows affected, 1 warning (0.01 sec)
```

### 3.7 用户和权限管理

#### 3.7.1 用户管理

和用户相关的元数据数据库是mysql,其中包含了数据库系统授权表

```mysql
(root@localhost) [mysql]> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| db                        |
| engine_cost               |
| event                     |
| func                      |
| general_log               |
```

**用户账号的组成**

```mysql
'USERNAME'@'HOST'
@'HOST'表示主机名
如：user2@'192.168.1.%'
```

**创建用户**

```mysql
CREATE USER 'USERNAME'@'HOST' [IDENTIFIED BY 'password']；
```

**用户重命名：RENAME USER**

```mysql
RENAME USER old_user_name TO new_user_name;
```

**删除用户：**

```mysql
DROP USER 'USERNAME'@'HOST‘
```

**修改用户密码：**

```mysql
#方法1
SET PASSWORD FOR 'user'@'host' = PASSWORD(‘password');

#方法2
UPDATE mysql.user SET password=PASSWORD('password') WHERE clause;
#mariadb 10.3
update mysql.user set authentication_string=password('ubuntu') where
user='mage';

#此方法需要执行下面指令才能生效：
FLUSH PRIVILEGES;
```

注意：新版mysql中用户密码可以保存在mysql.user表的authentication_string字段中，而且authentication_string 优先生效

忘记管理员密码解决方法：

```mysql
#修改数据库配置文件
[root@centos8 ~]#vim /etc/my.cnf
[mysqld]
skip-grant-tables             #跳过用户密码认证                                    
          skip-networking		#防止远程无密码登陆数据库
[root@centos8 ~]#systemctl restart mariadb

#修改root用户密码
[root@centos8 ~]#mysql
MariaDB [(none)]> update mysql.user set authentication_string=password('ubuntu')
where user='root';

[root@centos8 ~]#systemctl restart mariadb

#改完注释掉选项
[root@centos8 ~]#vim /etc/my.cnf
[mysqld]
#skip-grant-tables                                                              
        #skip-networking
[root@centos8 ~]#systemctl restart mariadb

#测试是否能使用新密码连接
[root@centos8 ~]#mysql -uroot -pubuntu
```

##### 3.7.2 权限管理

```mysql
#格式
GRANT priv_type [(column_list)],... ON [object_type] priv_level TO 'user'@'host'
[IDENTIFIED BY 'password'] [WITH GRANT OPTION];

priv_type: ALL [PRIVILEGES]

object_type:TABLE | FUNCTION | PROCEDURE

priv_level: *(所有库) |*.*   | db_name.* | db_name.tbl_name | tbl_name(当前库
的表) | db_name.routine_name(指定库的函数,存储过程,触发器)
```

参考链接：https://dev.mysql.com/doc/refman/5.7/en/grant.html

范例：

```mysql
GRANT SELECT (col1), INSERT (col1,col2) ON mydb.mytbl TO 'someuser'@'somehost‘;

GRANT ALL ON wordpress.* TO wordpress@'192.168.8.%' IDENTIFIED BY 'database';

GRANT ALL PRIVILEGES ON *.* TO 'root'@'192.168.8.%' IDENTIFIED BY 'database'
WITH GRANT OPTION;
```

取消授权：REVOKE

```mysql
#格式
REVOKE priv_type [(column_list)] [, priv_type [(column_list)]] ... ON
[object_type] priv_level FROM user [, user] ...

REVOKE DELETE ON testdb.* FROM 'testuser'@‘172.16.0.%’;
```

参考链接：https://dev.mysql.com/doc/refman/5.7/en/revoke.html

查看用户的权限

```mysql
Help SHOW GRANTS
SHOW GRANTS FOR 'user'@'host';
SHOW GRANTS FOR CURRENT_USER[()];
```

## 4 MySql架构

#### 4.1 MySql架构

**MySql架构分层**

1）网络连接层

2）服务层

3）存储引擎层

4）系统文件层

[![image](https://img2020.cnblogs.com/blog/2222624/202106/2222624-20210628221846875-1728221388.png)](https://img2020.cnblogs.com/blog/2222624/202106/2222624-20210628221846875-1728221388.png)

**网络连接层：**

客户端连接器:提供与mysql服务器的连接，每种语言(java, c, php)通过自己的api技术维护与mysql服务器的连接

**服务层：**

- 连接池（Connection Pool）：负责存储和管理客户端与数据库的连接，一个线程负责管理一个连接

- 系统管理和控制工具：如备份恢复、安全管理、集群管理等

- SQL接口（Sql interface ）：用于接受客户端的各种SQl命令，例如DDL, DML，存储过程，视图，触发器等

- 解析器（Parser）：负责将请求的SQL解析成一个解析树，然后根据一些MySql规则进一步检查解析树是否合法

- 查询优化器（Query Optimizer）：在解析树通过解析器语法检查之后，查询优化器将其转换为执行计划，然后与存储引擎进行交互

  例如：select uid,name from user where gender=1;

  1）Select首先根据where语句选择，而不是查询所有数据然后过滤

  2）Select query基于uid和name列进行查询

  3）将两个查询条件联合起来生成查询结果

- 缓存（Cache）：缓存是由一系列小缓存组成。比如表缓存，记录缓存，权限缓存，引擎缓存，如果查询缓存中有命中的查询结果，查询语句就可以直接去查询缓存中取数据

**存储引擎：**

在MySQL中存储引擎负责存储和读取数据，并且和底层的文件系统交互。存储引擎是MySQL的插件，主流的两种存储引擎是MyISAM 和InnoDB

**文件系统层：**

包括日志文件、配置文件、数据文件、pid文件、socket文件

#### 4.2 存储引擎

MySQL中的数据用各种不同的技术存储在文件（或者内存）中。这些技术中的每一种技术都使用不同的 存储机制、索引技巧、锁定水平并且最终提供广泛的不同的功能和能力,此种技术称为存储擎,MySQL 支 持多种存储引擎其中目前应用最广泛的是InnoDB和MyISAM两种

官方参考资料：https://dev.mysql.com/doc/refman/8.0/en/storage-engines.html

##### 4.2.1 MyISAM存储引擎和 InnoDB引擎

**两种引擎主要特性**

|        特性         |   MyISAM   |      InnoDB      |
| :-----------------: | :--------: | :--------------: |
| 事务(transactions)  |   不支持   |       支持       |
|        MVCC         |   不支持   |       支持       |
| Locking granularity |   table    |       row        |
|      数据缓存       |   不支持   |       支持       |
|      索引缓存       |    支持    |       支持       |
|      聚簇索引       |   不支持   |       支持       |
|      读写阻塞       | 读时不能写 | 事务隔离级别相关 |

**文件**

|   文件   |    MyISAM    |    InnoDB    |
| :------: | :----------: | :----------: |
| 表格定义 | tbl_name.frm | tbl_name.frm |
| 数据文件 | tbl_name.MYD | tbl_name.idb |
| 索引文件 | tbl_name.MYI | tbl_name.idb |

每个表单独使用一个表空间存储表的数据和索引

```mysql
两类文件放在对应每个数据库独立目录中
数据文件(存储数据和索引)：tb_name.ibd
表格式定义：tb_name.frm
```

##### 4.2.2 存储引擎的管理

查看MySQL支持的存储引擎

```mysql
show engines	
```

查看默认的存储引擎

```mysql
show variables like '%storage_engine%';
```

设置默认的存储引擎

```mysql
vim /etc/my.cnf
[mysqld]
default_storage_engine= InnoDB	
```

查看库中所有表使用的存储引擎

```mysql
show tables status from db_name;
```

查看库中指定表的存储引擎

```mysql
show table status like 'tb_name';
show create table tb_name;	
```

设置表的存储引擎

```mysql
create table tb_name(...) engine=innodb;
alter table tb_name engine=innodb;
```

#### 4.3 Mysql中数据库和服务器配置

##### 4.3.1 MySQL中的数据库

初始的Mysql自带数据库

```mysql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

- MySQL

  是mysql的核心数据库，类似于Sql Server中的master库，主要负责存储数据库的用户、权限设 置、关键字等mysql自己需要使用的控制和管理信息

- performance_schema数据库

  主要用于收集数据库服务器性能参数,库里表的存储引擎均为 PERFORMANCE_SCHEMA，用户不能创建存储引擎为PERFORMANCE_SCHEMA的表

- sys数据库

  目标是把 performance_schema的把复杂度降低，让DBA能更好的阅读这个库里的内容。让DBA更快的了 解DB的运行情况

- information_schema数据库

  一个虚拟数据库，物理上并不存在information_schema数据库类似与“数 据字典”，提供了访问数据库元数据的方式，即数据的数据。比如数据库名或表名，列类型，访问 权限（更加细化的访问方式）

##### 4.3.2 服务器状态配置

官方文档：

https://dev.mysql.com/doc/refman/8.0/en/server-option-variable-reference.html

**服务器选项：**

```mysql
#查看mysqld可用选项列表和及当前值
[root@localhost mysql]# /app/mysql/bin/mysql --verbose --help

#获取mysqld当前启动选项
[root@localhost mysql]# mysqld --print-defaults
mysqld would have been started with the following arguments:
--datadir=/data/mysql --skip_name_resolve=1 --socket=/data/mysql/mysql.sock --log-error=/data/mysql/mysql.log --pid-file=/data/mysql/mysql.pid 
```

通常服务器选项有两种设置方式

```mysql
#1.在命令行中设置
[root@localhost mysql]# /app/mysql/bin/mysqld --basedir=/usr

#2.在配置文件中my.cnf设置
vim /etc/my.cnf
[mysqld]
skip_name_resolve=1
```

查看服务器变量

```mysql
SHOW GLOBAL VARIABLES; #只查看global变量
SHOW [SESSION] VARIABLES;#查看所有变量(包括global和session)

#查看指定的系统变量
SHOW VARIABLES LIKE 'VAR_NAME';
SELECT @@VAR_NAME;
```

修改变量

```mysql
#修改全局变量
SET GLOBAL system_var_name=value;
SET @@global.system_var_name=value;

#修改会话变量
SET [SESSION] system_var_name=value;
SET @@[session.]system_var_name=value;
```

#### 4.4 查询缓存

##### 4.4.1 查询缓存原理

[![image](https://img2020.cnblogs.com/blog/2222624/202106/2222624-20210628221957116-1884186356.png)](https://img2020.cnblogs.com/blog/2222624/202106/2222624-20210628221957116-1884186356.png)

缓存SELECT操作或预处理查询的结果集和SQL语句，当有新的SELECT语句或预处理查询语句请求，先 去查询缓存，判断是否存在可用的记录集，判断标准：与缓存的SQL语句，是否完全一样，区分大小写

**查询缓存相关的服务器变量**

- query_cache_type：是否开启缓存功能，取值为ON, OFF, DEMAND
- query_cache_size：查询缓存总共可用的内存空间；单位字节，必须是1024的整数倍，最小值 40KB，低于此值有警报，当此值为0时也表示未开启查询缓存

更多选项参考官方文档：

https://dev.mysql.com/doc/refman/5.7/en/query-cache.html

```mysql
mysql> show variables like '%query_cache%';
+------------------------------+---------+
| Variable_name                | Value   |
+------------------------------+---------+
| have_query_cache             | YES     |
| query_cache_limit            | 1048576 |
```

**查询缓存相关的状态变量**

```mysql
mysql> SHOW GLOBAL STATUS LIKE 'Qcache%';
+-------------------------+---------+
| Variable_name           | Value   |
+-------------------------+---------+
| Qcache_free_blocks      | 1       |
| Qcache_free_memory      | 1031832 |
| Qcache_hits             | 0       |
| Qcache_inserts          | 0       |
| Qcache_lowmem_prunes    | 0       |
```

##### 4.4.2 MySQL 8.0 变化

MySQL8.0 取消查询缓存的功能

有时因为查询缓存往往弊大于利。比如:查询缓存的失效非常频繁，只要有对一个表的更新，这个表 上的所有的查询缓存都会被清空。因此很可能你费劲地把结果存起来，还没使用呢，就被一个更新全清 空了。对于更新压力大的数据库来说，查询缓存的命中率会非常低。除非你的业务有一张静态表，很长 时间更新一次，比如系统配置表，那么这张表的查询才适合做查询缓存。

目前大多数应用都把缓存做到了应用逻辑层，比如:使用redis或者memcache

#### 4.5 INDEX索引

索引：是排序的快速查找的特殊数据结构，定义作为查找条件的字段上，又称为键key，就像是每本书的目录索引通过存储 引擎实现，有了索引能以降低服务需要扫描的数据量，减少了IO次数，提高查询语句执行的速度

##### 4.5.1 索引结构

**B+Tree索引**

[![image](https://img2020.cnblogs.com/blog/2222624/202106/2222624-20210628222026305-760459483.jpg)](https://img2020.cnblogs.com/blog/2222624/202106/2222624-20210628222026305-760459483.jpg)

B+Tree索引：按顺序存储，每一个叶子节点到根结点的距离是相同的；左前缀索引，适合查询范围类的 数据

B-tree和B+Tree唯一不同之处就是B+Tree是将数据存储在页上

**B+Tree索引的限制：**

- 如不从最左列开始，则无法使用索引，如：查找名为huanhuan，或姓为g结尾
- 不能跳过索引中的列：如：查找姓li，年龄30的，只能使用索引第一列

**Hash索引**

Hash索引：基于哈希表实现，只有精确匹配索引中的所有列的查询才有效，索引自身只存储索引列对 应的哈希值和数据指针，索引结构紧凑，查询性能好

**聚簇和非聚簇索引**

- 聚簇索引：将数据存储与索引放到了一块，找到索引也就找到了数据
- 非聚簇索引：将数据存储于索引分开结构，索引结构的叶子节点指向了数据的对应行，myisam通过key_buffer把索引先缓存到内存中，当需要访问数据时（通过索引访问数据），在内存中直接搜索索引，然后通过索引找到磁盘相应数据，这也就是为什么索引不在key buffer命中时，速度慢的原因

##### 4.5.2 索引优化

- 在where条件中，始终将索引列单独放在比较符号的一侧，尽量不要在列上进行运算（函数 操作和表达式操作）
- 左前缀索引：构建指定索引字段的左侧的字符数，要通过索引选择性（不重复的索引值和数据表的 记录总数的比值）来评估，尽量使用短索引，如果可以，应该制定一个前缀长度
- 多列索引：AND操作时更适合使用多列索引，而非为每个列创建单独的索引
- 选择合适的索引列顺序：无排序和分组时，将选择性最高放左侧
- 只要列中含有NULL值，就最好不要在此列设置索引，复合索引如果有NULL值，此列在使用时也 不会使用索引
- 对于like语句，以 % 或者 _ 开头的不会使用索引，以 % 结尾会使用索引
- 对于经常在where子句使用的列，最好设置索引
- 对于有多个列where或者order by子句，应该建立复合索引
- 尽量不要使用not in和<>操作,虽然可能使用索引,但性能不高

官方文档：https://dev.mysql.com/doc/refman/8.0/en/optimization-indexes.html

##### 4.5.3 管理索引

创建索引

```mysql
CREATE [UNIQUE | FULLTEXT | SPATIAL] INDEX index_name
    [index_type]
    ON tbl_name (key_part,...)
    [index_option]
    [algorithm_option | lock_option] ...CREATE [UNIQUE | FULLTEXT | SPATIAL] INDEX index_name
    [index_type]
    ON tbl_name (key_part,...)
    [index_option]
    [algorithm_option | lock_option] ...
```

删除索引

```mysql
DROP INDEX index_name ON tbl_name;
ALTER TABLE tbl_name DROP INDEX index_name(index_col_name);
```

查看索引

```mysql
SHOW INDEXES FROM [db_name.]tbl_name;
```

优化表空间：

```mysql
#用于删除数据库后仍然占用空间情况
OPTIMIZE TABLE tb_name;
```

##### 4.5.4 EXPLAIN 工具

可以通过EXPLAIN来分析索引的有效性,获取查询执行计划信息，用来查看查询优化器如何执行查询

官方文档：https://dev.mysql.com/doc/refman/8.0/en/using-explain.html

```mysql
EXPLAIN SELECT clause
```

EXPLAN输出格式说明： type显示的是访问类型，是较为重要的一个指标，结果值从好到坏依次是：system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL ，一般来说，得保证查询至少达到range级别，最好能达到ref

#### 4.6 并发控制

##### 4.6.1 锁机制

锁：

- 读锁：共享锁，也称为 S 锁,只读不可写（包括当前事务） ，多个读互不阻塞
- 写锁：独占锁，排它锁，也称为 X 锁,写锁会阻塞其它事务（不包括当前事务）的读和写

锁粒度：

- 表级锁：MyISAM
- 行级锁：InnodB

分类：

- 隐式锁：由存储引擎自动施加锁
- 显式锁：用户手动请求

显示锁的使用：

官方文档：https://dev.mysql.com/doc/refman/8.0/en/lock-tables.html

##### 4.6.2 事务

**1、事务简介**

事务Transactions：一组原子性的SQL语句，或一个独立工作单元

事务的作用：保证所有事务都作为⼀个⼯作单元来执⾏，即使出现了故障，都不能改变 这种执⾏⽅式。当在⼀个事务中执⾏多个操作时，要么所有的事务都被提交(commit)，那么这些修改 就永久地保存下来；要么数据库管理系统将放弃所作的所有修改，整个事务回滚(rollback)到最初状态。最常见的例子就是银行卡转账，有了事务就可以防止转出和转入金额不一致情况。

**2、事务的ACID属性：**

- A：atomicity原子性；也就是不可分割，整个事务要么全都执行，要么都不执行
- C：consistency一致性；数据库总是从一个一致性状态转换为另一个一致性状态
- I：Isolation隔离性；一个事务所做出的操作在提交之前，是不能为其它事务所见；隔离有多种隔 离级别，实现并发
- D：durability持久性；一旦事务提交，其所做的修改会永久保存于数据库中

**3、事务管理**

官方文档：https://dev.mysql.com/doc/refman/8.0/en/commit.html

显示启动事务

```mysql
BEGIN
BEGIN WORK
START TRANSACTION
```

结束事务：

```mysql
#提交
COMMIT
#回滚
ROLLBACK
```

注意：DDL语句是不支持ROLLBACK的

在MySQL中事务默认是自动提交的，也可以修改这个选项

```mysql
set autocommit={1|0} 
默认为1，为0时设为非自动提交
若设置为0表示我们的DML操作都需使用commit提交后才会生效
```

事务还支持设置保存点，类似于虚拟机中的快照

```mysql
SAVEPOINT identifier
ROLLBACK [WORK] TO [SAVEPOINT] identifier
RELEASE SAVEPOINT identifier
```

查看当前事务：

```mysql
#查看当前正在进行的事务
SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX;

#查看当前锁定的事务
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;

#查看当前等锁的事务
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;
```

**4、事务的隔离级别**

数据库事务的隔离性：数据库系统必须具有隔离并发运⾏各个事务的能⼒, 使它们不会相互影响, 避免各 种并发问题。⼀个事务与其他事务隔离的程度称为隔离级别。数据库规定了多种事务隔离级别, 不同隔 离级别对应不同的⼲扰程度, 隔离级别越⾼, 数据⼀致性就越好, 但并发性越弱。

数据库提供的 4 种事务隔离级别：

| 隔离级别         |                                                              |
| :--------------- | :----------------------------------------------------------- |
| read-uncommitted | 允许事务读取其他事务未提交和已提交的数据。会出现脏读、不可重复读、 幻读 问题 |
| read-committed   | 只允许事务读取其他事务已提交的数据。可以避免脏读，但仍然会出现不可重复读、幻读问题 |
| repeatable-read  | 确保事务可以多次从⼀个字段中读取相同的值。在这个事务持续期间，禁⽌其 他事务对这个字段进⾏更新。可以避免脏读和不可重复读。但是幻读问题仍然 存在。 |
| serializable     | 确保事务可以从⼀个表中读取相同的⾏，相同的记录。在这个事务持续期间， 禁⽌其他事务对该表执⾏插⼊、更新、删除操作。所有并发问题都可以避免， 但性能⼗分低下。 |

注意：在mysql 中REPEATABLE READ的隔离级别也可以避免幻读。可以在官方文档看到详细说明https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html

MVCC（多版本并发控制机制）只在REPEATABLE READ和READ COMMITTED两个隔离级别下工作。其他两个隔离级别都和MVCC不兼容,因为READ UNCOMMITTED总是读取最新的数据行，而不是符合当前事务版本的数据行。而SERIALIZABLE则会对所有读取的行都加锁

指定事务隔离级别：

- 服务器变量tx_isolation指定，默认为REPEATABLE-READ，可在GLOBAL和SESSION级进行设置

  ```mysql
  SET tx_isolation='READ-UNCOMMITTED|READ-COMMITTED|REPEATABLEREAD|SERIALIZABLE'
  ```

- 服务器选项中指定

  ```mysql
  vim /etc/my.cnf
  [mysqld]
  transaction-isolation=SERIALIZABLE
  ```

#### 4.7 日志管理

##### 4.7.1 事务日志

事务日志记录了所有的事务操作，包括未提交的事务，而且它是先于数据库数据的

事务日志分为Redo Log和Undo Logs，它是由事务型存储引擎自行管理和使用

官方文档：https://dev.mysql.com/doc/refman/8.0/en/innodb-redo-log.html

**事务日志性能优化**

```mysql
innodb_flush_log_at_trx_commit=0|1|2

1 此为默认值，日志缓冲区将写入日志文件，并在每次事务后执行刷新到磁盘。 这是完全遵守ACID特性

0 提交时没有写磁盘的操作; 而是每秒执行一次将日志缓冲区的提交的事务写入刷新到磁盘。 这样可提供
更好的性能，但服务器崩溃可能丢失最后一秒的事务

2 每次提交后都会写入OS的缓冲区，但每秒才会进行一次刷新到磁盘文件中。 性能比0略差一些，但操作系
统或停电可能导致最后一秒的交易丢失
```

因此0和2模式性能最好，而1模式更加安全，一般为了性能考虑我们会选择2模式

```mysql
1.配置为2和配置为0，性能差异并不大，因为将数据从Log Buffer拷贝到OS cache，虽然跨越用户态与内
核态，但毕竟只是内存的数据拷贝，速度很快

2.配置为2和配置为0，安全性差异巨大，操作系统崩溃的概率相比MySQL应用程序崩溃的概率，小很多，设置
为2，只要操作系统不奔溃，也绝对不会丢数据
```

设置为1，同时sync_binlog = 1表示最高级别的容错

其他相关变量

```
#使用show variables like '%innodb_log%';查看

innodb_log_file_size   50331648 每个日志文件大小
innodb_log_files_in_group  2     日志组成员个数
innodb_log_group_home_dir ./ 事务文件路径
innodb_flush_log_at_trx_commit 默认为1
```

官方文档：https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_log_group_home_dir

##### 4.7.2 错误日志

日志官方文档链接：https://dev.mysql.com/doc/refman/8.0/en/server-logs.html

错误日志主要记录的是mysqld启动和关闭过程中输出的事件信息，运行过程中产生的错误信息

查看错误日志文件路径

```mysql
mysql> show global variables like 'log_error';
+---------------+-----------------------+
| Variable_name | Value                 |
+---------------+-----------------------+
| log_error     | /data/mysql/mysql.log |
+---------------+-----------------------+
1 row in set (0.01 sec)
```

记录哪些级别的日志

```mysql
mysql>  SHOW GLOBAL VARIABLES LIKE 'log_error_verbosity';
+---------------------+-------+
| Variable_name       | Value |
+---------------------+-------+
| log_error_verbosity | 3     |
+---------------------+-------+
1 row in set (0.01 sec)
```

官方文档：https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_log_error_verbosity

##### 4.7.3 通用日志

记录对数据库的通用操作，包括:错误的SQL语句

通用日志相关设置

```mysql
#是否开启通用日志
general_log=ON|OFF

#通用日志文件位置
general_log_file=HOSTNAME.log

#通用日志保存位置（文件或者表）
log_output=TABLE|FILE|NONE
```

##### 4.7.4 慢查询日志

记录执行查询时长超出指定时长的操作

慢查询相关变量

```mysql
slow_query_log=ON|OFF #开启或关闭慢查询，支持全局和会话，只有全局设置才会生成慢查询文件
long_query_time=N #慢查询的阀值，单位秒
slow_query_log_file=HOSTNAME-slow.log  #慢查询日志文件
log_slow_filter = admin,filesort,filesort_on_disk,full_join,full_scan,
query_cache,query_cache_miss,tmp_table,tmp_table_on_disk
#上述查询类型且查询时长超过long_query_time，则记录日志
log_queries_not_using_indexes=ON  #不使用索引或使用全索引扫描，不论是否达到慢查询阀值的语
句是否记录日志，默认OFF，即不记录
log_slow_rate_limit = 1 #多少次查询才记录，mariadb特有
log_slow_verbosity= Query_plan,explain #记录内容
log_slow_queries = OFF    #同slow_query_log，MariaDB 10.0/MySQL 5.6.1 版后已删除
```

日志查看

```mysql
#可以打开日志文件直接查看
[root@localhost mysql]# cat /data/mysql/localhost-slow.log 
/app/mysql/bin/mysqld, Version: 5.7.34 (Source distribution). started with:
Tcp port: 3306  Unix socket: /data/mysql/mysql.sock
Time                 Id Command    Argument
# Time: 2021-06-18T17:03:12.310465Z
# User@Host: root[root] @ localhost []  Id:     2

#日志分析工具查看
[root@localhost mysql]# mysqldumpslow -s c -t 10 /data/mysql/localhost-slow.log 

Reading mysql slow query log from /data/mysql/localhost-slow.log
Count: 1  Time=0.00s (0s)  Lock=0.00s (0s)  Rows=0.0 (0), 0users@0hosts
 
```

官方文档：https://dev.mysql.com/doc/refman/8.0/en/mysqldumpslow.html

##### 4.7.5 profile工具

可以显示语句的执行详细过程

启用profile

```mysql
mysql> set profiling=1;
Query OK, 0 rows affected, 1 warning (0.01 sec)
```

查看语句执行过程

```mysql
#查看执行的最新语句列表
mysql> show profiles;
+----------+------------+------------------------+
| Query_ID | Duration   | Query                  |
+----------+------------+------------------------+
|        1 | 0.00040025 | select * from teachers |
+----------+------------+------------------------+
1 row in set, 1 warning (0.00 sec)

#查看语句执行的详细过程
mysql> show profile for query 1;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000092 |
| checking permissions | 0.000010 |
```

##### 4.7.6 二进制日志

描述数据库更改的“事件”，例如表创建操作或表数据的更改。以及潜在导致数据库改变的SQL语句，二进制日志还有连个重要作用：

- 复制
- 备份，通过“重放”日志文件中的事件来生成数据副本

建议将二进制日志和数据文件分开存放

**二进制日志三种记录格式**

- 基于“语句”记录：statement，记录语句
- 基于“行”记录：row，记录数据，日志量较大，更加安全，建议使用的格式
- 混合模式：mixed, 让系统自行判定该基于哪种方式进行

官方文档：https://dev.mysql.com/doc/refman/8.0/en/binary-log.html

**格式配置**

```mysql
mysql>  show variables like 'binlog_format';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| binlog_format | ROW   |
+---------------+-------+
1 row in set (0.00 sec)
```

**二进制日志相关的服务器变量：**

```mysql
sql_log_bin=ON|OFF：#是否记录二进制日志，默认ON，支持动态修改，系统变量，而非服务器选项

log_bin=/PATH/BIN_LOG_FILE：#指定文件位置；默认OFF，表示不启用二进制日志功能，上述两项都开
启才可以而且这个选项只能在配置文件中修改
#在MySQL中还需指定server_id系统变量，否则不能启动MySQL

binlog_format=STATEMENT|ROW|MIXED：#二进制日志记录的格式，默认STATEMENT

max_binlog_size=1073741824：#单个二进制日志文件的最大体积，到达最大值会自动滚动，默认为1G

#说明：文件达到上限时的大小未必为指定的精确值
binlog_cache_size=4m #此变量确定在每次事务中保存二进制日志更改记录的缓存的大小（每次连接）

max_binlog_cache_size=512m #限制用于缓存多事务查询的字节大小。

sync_binlog=1|0：#设定是否启动二进制日志即时同步磁盘功能，默认0，由操作系统负责同步日志到磁盘

expire_logs_days=N：#二进制日志可以自动删除的天数。 默认为0，即不自动删除
```

**二进制日志文件组成**

```mysql
有两类文件
1.日志文件：mysql|mariadb-bin.文件名后缀，二进制格式,如： mariadb-bin.000001
2.索引文件：mysql|mariadb-bin.index，文本格式
```

**二进制日志语句**

```mysql
#查看二进制日志文件列表
mysql> show binary logs;
+------------------+-----------+
| Log_name         | File_size |
+------------------+-----------+
| mysql-bin.000001 |       154 |
+------------------+-----------+
1 row in set (0.00 sec)

#查看使用中的二进制文件
mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000001 |      154 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)

#查看指定二进制文件中的指定内容
SHOW BINLOG EVENTS [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count]
show binlog events in 'mysql-bin.000001' from 6516 limit 2,3
```

**离线查看二进制日志**

官方文档：https://dev.mysql.com/doc/refman/8.0/en/mysqlbinlog.html

```mysql
#二进制日志的客户端命令工具，支持离线查看二进制日志
mysqlbinlog [OPTIONS] log_file…
 --start-position=# 指定开始位置
 --stop-position=#
 --start-datetime=  #时间格式：YYYY-MM-DD hh:mm:ss
 --stop-datetime=
 --base64-output[=name]
        -v -vvv
        
 [root@localhost mysql]# mysqlbinlog -v /data/mysql-bin.000001
/*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
/*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
DELIMITER /*!*/;
# at 4
#210619  2:51:55 server id 0  end_log_pos 123 CRC32 0x83172d33 	Start: binlog v 4, server v 5.7.34-log created 210619  2:51:55 at startup
# Warning: this binlog is either in use or was not closed properly.
ROLLBACK/*!*/;
BINLOG '
```

**清除指定二进制日志**

```mysql
PURGE { BINARY | MASTER } LOGS { TO 'log_name' | BEFORE datetime_expr }

PURGE BINARY LOGS TO 'mariadb-bin.000003'; #删除mariadb-bin.000003之前的日志
PURGE BINARY LOGS BEFORE '2017-01-23';
PURGE BINARY LOGS BEFORE '2017-03-22 09:25:30';
```

**删除所有二进制日志，index文件重新计数**

```mysql
RESET MASTER [TO #]; #删除所有二进制日志文件，并重新生成日志文件，文件名从#开始记数，默认从
1开始，一般是master主机第一次启动时执行，MariaDB 10.1.6开始支持TO #
```

生成新日志文件；

```mysql
FLUSH LOGS;
```

## 5 备份和恢复

### 5.1 概述

备份主要就是为了防止数据丢失

#### 5.1.1 备份类型

完全备份：整个数据集

部分备份：只备份数据子集，如部分库或表

增量备份：仅备份最近一次完全备份或增量备份（如果存在增量）以来变化的数据，备份较快，还原复杂

差异备份：仅备份最近一次完全备份以来变化的数据，备份较慢，还原简单

冷、温、热备份

- 冷备：读、写操作均不可进行，数据库停止服务
- 温备：读操作可执行；但写操作不可执行
- 热备：读、写操作均可执行
- MyISAM：温备，不支持热备而InnoDB：都支持

物理备份和逻辑备份

- 物理备份：直接复制数据文件进行备份，与存储引擎有关，占用较多的空间，速度快
- 逻辑备份：从数据库中“导出”数据另存而进行的备份，与存储引擎无关，占用空间少，速度慢，可 能丢失精度

##### 5.1.2 备份工具

备份内容：数据、二进制日志和配置文件

- cp, tar等复制归档工具：物理备份工具
- LVM的快照：先加读锁，做快照后解锁，几乎热备；不过需要使用LVM
- mysqldump：逻辑备份工具，适用所有存储引擎
- xtrabackup：由Percona提供支持对InnoDB做热备(物理备份)的工具，支持完全备份、增量备份
- MariaDB Backup： 从MariaDB 10.1.26开始集成，基于Percona XtraBackup 2.3.8实现

范例：实现MySQL冷备份

```mysql
源主机：10.0.0.2
备份主机：10.0.0.3
#两个主机安装数据库，注意备份主机不启动服务
[root@centos8 ~]# yum -y install mysql-server

#10.0.0.2步骤
[root@centos8 ~]# systemctl stop mysqld
#复制配置文件
[root@centos8 ~]# rsync -av /etc/my.cnf.d/mysql-server.cnf 10.0.0.3:/etc/my.cnf.d/
#打包并复制数据库文件
[root@centos8 ~]# cd /var/lib/mysql
[root@centos8 mysql]# tar zcvf mysql.tar.gz *
#拷贝数据至备份主机
[root@centos8 mysql]# scp mysql.tar.gz 10.0.0.3:/data
#拷贝二进制日志至备份主机
[root@centos8 mysql]# rsync -av /data/ 10.0.0.3:/data

#10.0.0.3步骤
#将文件解压缩至数据库目录
[root@centos8 ~]# cd /var/lib/mysql
[root@centos8 mysql]# tar xvf mysql.tar.gz
#修改数据库权限
[root@centos8 mysql]# chown -R mysql.mysql /var/lib/mysql
[root@centos8 mysql]# chown -R mysql.mysql /data/
#启动数据库
[root@centos8 ~]# systemctl start mysqld
```

### 5.2 mysqldump备份

#### 5.2.1 mysqldump 说明

**逻辑备份工具：**

通过SQL语句来复现原表，达到逻辑备份其命令格式如下：

```mysql
mysqldump [OPTIONS] database [tables]   #支持指定数据库和指定多表的备份，但数据库本身定
义不备份
mysqldump [OPTIONS] –B DB1 [DB2 DB3...] #支持指定数据库备份，包含数据库本身定义也会备份
mysqldump [OPTIONS] –A [OPTIONS]        #备份所有数据库，包含数据库本身定义也会备份
```

参考文档：https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html

mysqldump常见选项说明：

```mysql
-A, --all-databases #备份所有数据库，含create database
-B, --databases db_name…  #指定备份的数据库，包括create database语句
-E, --events：#备份相关的所有event scheduler
-R, --routines：#备份所有存储过程和自定义函数
--triggers：#备份表相关触发器，默认启用,用--skip-triggers，不备份触发器
--default-character-set=utf8 #指定字符集
--master-data[=#]： #此选项须启用二进制日志
#1：所备份的数据之前加一条记录为CHANGE MASTER TO语句，非注释，不指定#，默认为1，适合于主从复制多机使用
#2：记录为被注释的#CHANGE MASTER TO语句，适合于单机使用
#此选项会自动关闭--lock-tables功能，自动打开-x | --lock-all-tables功能（除非开启--single-transaction）
-F, --flush-logs #备份前滚动日志，锁定表完成后，执行flush logs命令,生成新的二进制日志文件，配合-A 或 -B 选项时，会导致刷新多次数据库。建议在同一时刻执行转储和日志刷新，可通过和--
single-transaction或-x，--master-data 一起使用实现，此时只刷新一次二进制日志
--compact #去掉注释，适合调试，生产不使用
-d, --no-data #只备份表结构,不备份数据
-t, --no-create-info #只备份数据,不备份表结构,即create table
-n,--no-create-db #不备份create database，可被-A或-B覆盖
--flush-privileges #备份mysql或相关时需要使用
-f, --force       #忽略SQL错误，继续执行
--hex-blob        #使用十六进制符号转储二进制列，当有包括BINARY， VARBINARY，
BLOB，BIT的数据类型的列时使用，避免乱码
-q, --quick     #不缓存查询，直接输出，加快备份速度
```

InnoDB建议备份策略

```mysql
mysqldump –uroot -p –A –F –E –R --triggers --single-transaction --master-data=1
--flush-privileges --default-character-set=utf8 --hex-blob
>${BACKUP}/fullbak_${BACKUP_TIME}.sql
```

MyISAM建议备份策略

```mysql
mysqldump –uroot -p –A –F –E –R –x --master-data=1 --flush-privileges --
triggers --default-character-set=utf8 --hex-blob
>${BACKUP}/fullbak_${BACKUP_TIME}.sql
```

范例：：每天2：30做完全备份，早上10：00误删除students，10：10才发现故障，现需要将数据 库还原到10：10的状态，且恢复被删除的students表

```mysql
#2点30开始完全备份，注意该实验中由于可以使用口令登陆所以没有指定用户和密码，实际生产中应添加-u和-p选项
[root@centos8 mysql]# mysqldump -A -F -E -R --triggers --single-transaction --master-data=2 | gzip > /backup/all-`date +%F`.sql.gz
#查看是否备份成功
[root@centos8 mysql]# ll /backup/
total 228
-rw-r--r-- 1 root root 232381 Jun 24 01:24 all-2021-06-24.sql.gz

#备份后的数据更新
mysql> insert students (name,age,gender) values('rose',20,'f');
Query OK, 1 row affected (0.02 sec)

mysql> insert students (name,age,gender) values('jack',22,'M');
Query OK, 1 row affected (0.02 sec

#10：00误删除表格
mysql> drop table students;
Query OK, 0 rows affected (0.14 sec)

#其他表继续发生更新
mysql> insert teachers (name,age,gender)values('sun',25,'M');
Query OK, 1 row affected (0.03 sec)

mysql> insert teachers (name,age,gender)values('ma',30,'M');
Query OK, 1 row affected (0.02 sec)

mysql> select * from teachers;
+-----+---------------+-----+--------+
| TID | Name          | Age | Gender |
+-----+---------------+-----+--------+
|   1 | Song Jiang    |  45 | M      |
|   2 | Zhang Sanfeng |  94 | M      |
|   3 | Miejue Shitai |  77 | F      |
|   4 | Lin Chaoying  |  93 | F      |
|   5 | sun           |  25 | M      |
|   6 | ma            |  30 | M      |
+-----+---------------+-----+--------+
6 rows in set (0.01 sec)

#10：10发现表删除，准备进行还原，停止数据库访问
[root@centos8 backup]# gunzip all-2021-06-24.sql.gz
[root@centos8 backup]# ll
total 1056   -rw-r--r-- 1 root root 1080060 Jun 24 01:24 all-2021-06-24.sql

#找到完全备份二进制位置
[root@centos8 backup]# grep '\-\- CHANGE MASTER TO' all-2021-06-24.sql 
-- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000007', MASTER_LOG_POS=156;

#备份从完全备份后的二进制日志
 [root@centos8 backup]# mysqlbinlog --start-position=156 /data/mysql-bin.000007 > /backup/inc.sql
[root@centos8 backup]# ll
total 1064
-rw-r--r-- 1 root root 1080060 Jun 24 01:24 all-2021-06-24.sql
-rw-r--r-- 1 root root    7724 Jun 24 02:07 inc.sql

#从备份中删除删表的语句
[root@centos8 ~]#vim /data/inc.sql
#DROP TABLE `student_info` /* generated by server */
#如果文件过大，可以使用sed实现
[root@centos8 ~]#sed -i.bak '/^DROP TABLE/d' /data/inc.sql
                          
#利用完全备份和修改过的二进制日志进行还原
mysql> source /backup/all-2021-06-24.sql
mysql> source /backup/inc.sql
mysql> set sql_log_bin=1;
```

### 5.3 XtraBackup

percona提供的mysql数据库备份工具，惟一开源的能够对innodb和xtradb数据库进行热备的工具

**xtrabackup版本说明**

xtrabackup版本升级到2.4后，相比之前的2.1有了比较大的变化：innobackupex 功能全部集成到 xtrabackup 里面，只有一个 binary程序，另外为了兼容考虑，innobackupex作为 xtrabackup 的软链 接，即xtrabackup现在支持非Innodb表备份，并且 Innobackupex 在下一版本中移除，建议通过 xtrabackup替换innobackupex

官方文档：https://www.percona.com/doc/percona-xtrabackup/LATEST/index.html

8.0版本下载地址：https://downloads.percona.com/downloads/Percona-XtraBackup-LATEST/Percona-XtraBackup-8.0.25-17/binary/redhat/8/x86_64/percona-xtrabackup-80-8.0.25-17.1.el8.x86_64.rpm

**xtrabackup备份过程**

[![image](https://img2020.cnblogs.com/blog/2222624/202106/2222624-20210628222108963-237634333.png)](https://img2020.cnblogs.com/blog/2222624/202106/2222624-20210628222108963-237634333.png)

**备份生成的相关文件**

官方文档：https://www.percona.com/doc/percona-xtrabackup/8.0/xtrabackup-files.html

- xtrabackup_info：文本文件，innobackupex工具执行时的相关信息，包括版本，备份选项，备 份时长，备份LSN(log sequence number日志序列号)，BINLOG的位置
- xtrabackup_checkpoints：文本文件，备份类型（如完全或增量）、备份状态（如是否已经为 prepared状态）和LSN范围信息,每个InnoDB页(通常为16k大小)都会包含一个日志序列号LSN。 LSN是整个数据库系统的系统版本号，每个页面相关的LSN能够表明此页面最近是如何发生改变的
- xtrabackup_binlog_info：文本文件，MySQL服务器当前正在使用的二进制日志文件及至备份这 一刻为止二进制日志事件的位置，可利用实现基于binlog的恢复
- backup-my.cnf：文本文件，备份命令用到的配置选项信息
- xtrabackup_logfile：备份生成的二进制日志文件

#### 5.3.1 xtrabackup用法

xtrabackup工具备份三部曲

- 备份：对数据库做完全或增量备份
- 预准备： 还原前，先对备份的数据，整理至一个临时目录，
- 还原：将整理好的数据，复制回数据库目录中

xtrabackup 选项参考

https://www.percona.com/doc/percona-xtrabackup/LATEST/xtrabackup_bin/xbk_option_reference.html

格式：

```mysql
xtrabackup [--defaults-file=#] --backup|--prepare|--copy-back|--stats [OPTIONS]
--user：#该选项表示备份账号
--password：#该选项表示备份的密码
--host：#该选项表示备份数据库的地址
--databases：#该选项接受的参数为数据库名，如果要指定多个数据库，彼此间需要以空格隔开；
如："xtra_test dba_test"，同时，在指定某数据库时，也可以只指定其中的某张表。
如："mydatabase.mytable"。该选项对innodb引擎表无效，还是会备份所有innodb表
--defaults-file：#该选项指定从哪个文件读取MySQL配置，必须放在命令行第一个选项位置
--incremental：#该选项表示创建一个增量备份，需要指定--incremental-basedir
--incremental-basedir：#该选项指定为前一次全备份或增量备份的目录，与--incremental同时使用
--incremental-dir：#该选项表示还原时增量备份的目录
--include=name：#指定表名，格式：databasename.tablename
```

范例：新版 xtrabackup完全备份及还原

```mysql
实验环境
Percona XtraBackup 8.0.25
Mysql版本8.0.21
[root@centos8 backup]# cat /etc/redhat-release 
CentOS Linux release 8.2.2004 (Core) 

1、安装软件包
[root@centos8 opt]# ls
percona-xtrabackup-80-8.0.25-17.1.el8.x86_64.rpm
[root@centos8 opt]# yum localinstall percona-xtrabackup-80-8.0.25-17.1.el8.x86_64.rpm 

2、在原主机做完全备份到/backup
#/backup目录不需事先创建，注意若mysql登陆若需要密码要加上-p选项
[root@centos8 opt]# xtrabackup -uroot --backup --target-dir=/backup/
.....
210628 19:48:23 completed OK!
#拷贝备份文件至还原主机
[root@centos8 opt]# scp -r /backup/* 10.0.0.3:/backup/
root@10.0.0.3's password: 
all-2021-06-24.sql                     100% 1055KB   6.0MB/s   00:00 

3、在目标主机上还原
#预准备：确保数据一致，提交完成的事务，回滚未完成的事务
[root@centos8 ~]# xtrabackup --prepare --target-dir=/backup/
.....
210628 11:55:42 completed OK!
#复制到数据库目录,数据库目录必须为空，MySQL服务不能启动
[root@centos8 ~]# xtrabackup --copy-back --target-dir=/backup/
.....
210628 12:27:27 [01] ...done.
210628 12:27:27 completed OK!
#修改属性
[root@centos8 ~]# chown -R mysql:mysql /var/lib/mysql
[root@centos8 ~]# systemctl start mysqld
#确认服务是否启动
[root@centos8 ~]# ss -ntl
State         Recv-Q        Send-Q               Local Address:Port                Peer Address:Port       
LISTEN        0             128                        0.0.0.0:22                       0.0.0.0:*          
LISTEN        0             70                               *:33060                          *:*          
LISTEN        0             128                              *:3306                           *:*          
LISTEN        0             128                           [::]:22                          [::]:*  
```

范例：利用xtrabackup完全，增量备份及还原

```mysql
实验环境与上例全量备份相同
1、备份
#完全备份
[root@centos8 ~]#xtrabackup -uroot --backup --target-dir=/backup/base
#第一次数据修改和增量备份
[root@centos8 ~]#xtrabackup -uroot --backup --target-dir=/backup/inc1 --incremental-basedir=/backup/base
#第二次数据修改和增量备份
[root@centos8 ~]#xtrabackup -uroot --backup --target-dir=/backup/inc2 --incremental-basedir=/backup/inc1
#查看备份文件夹
[root@centos8 backup]# ls /backup/
base  inc1  inc2
#拷贝文件至备份主机
[root@centos8 backup]# scp -r /backup/* 10.0.0.3:/backup/

2、还原
#准备全量备份文件，注意使用--apply-log-only 阻止回滚未完成的事务
[root@centos8 ~]# xtrabackup --prepare --apply-log-only --target-dir=/backup/base
#合并第1次增量备份到完全备份
[root@centos8 ~]# xtrabackup --prepare --apply-log-only --target-dir=/backup/base --incremental-dir=/backup/inc1
#合并第2次增量备份到完全备份，注意最后一次还原不需要加选项--apply-log-only
[root@centos8 ~]# xtrabackup --prepare --target-dir=/backup/base --incremental-dir=/backup/inc2
#复制到数据库目录
[root@centos8 ~]# xtrabackup --copy-back --target-dir=/backup/base
#修改文件属性
[root@centos8 ~]#chown -R mysql:mysql /var/lib/mysql
#启动服务
[root@centos8 ~]# systemctl start mysqld
```

范例：xtrabackup单表导出和导入

参考链接：https://www.percona.com/doc/percona-xtrabackup/8.0/xtrabackup_bin/partial_backups.html

```mysql
#备份单个表
[root@centos8 backup]# xtrabackup --backup --datadir=/var/lib/mysql --target-dir=/backups/ --tables="db1.export_test"
#备份表结构
[root@centos8 /]# mysql -e 'show create table db1.export_test' > /backups/student.sql
#拷贝文件至备份主机
[root@centos8 /]# scp -r /backups/* 10.0.0.3:/backup/

备份主机上执行
#导出单个表
[root@centos8 ~]# xtrabackup --prepare --export --target-dir=/backup/
#创建表
mysql> CREATE TABLE `export_test` (\n  `a` int DEFAULT NULL\n) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
PAGER set to stdout
PAGER set to stdout
Query OK, 0 rows affected (0.19 sec)
#删除表空间
mysql> alter table export_test discard tablespace;
Query OK, 0 rows affected (0.06 sec)
#拷贝导出的表到本地数据库目录
[root@centos8 ~]# cp /backup/db1/* /var/lib/mysql/db1/
#修改文件权限
[root@centos8 ~]# chown -R mysql.mysql /var/lib/mysql/db1/
#导入表空间
mysql> ALTER TABLE export_test IMPORT TABLESPACE;
Query OK, 0 rows affected (0.26 sec)

mysql> select * from export_test;
+------+
| a    |
+------+
|    1 |
|    2 |
+------+
3 rows in set (0.00 sec)
```

## 6 MySQL 集群 Cluster

### 6.1 MySQL主从复制

#### 6.1.1 主从复制架构和原理

**主从复制原理：**

主节点发生数据更新，并将更新的语句记录到二进制日志当中，主节点会启动一个dump线程，用于向其从节点发送binary log events。从节点会启动I/O Thread和SQL Thread两个线程，其中I/O Thread用于向Master请求二进制日志事件，并保存于中继日志中；SQL Thread则用于从中继日志中读取日志事件，在本地完成重放。

**常见的复制架构：**

- 主从复制
- 一主多从架构
- 一主一从带多从架构

#### 6.1.2 主从复制的实现

参考文档：https://dev.mysql.com/doc/refman/8.0/en/replication.html

**配置步骤：**

1）主节点配置

启用二进制日志

```mysql
#通过原理可知主从复制是以二进制日志为基础进行复制操作，所以主节点必须开启二进制日志
[mysqld]
log_bin
```

为当前节点设置一个全局惟一的ID号

```mysql
[mysqld]
server-id=#
log-basename=master  #可选项，设置datadir中日志名称，确保不依赖主机名
```

创建有复制权限的用户账号

```mysql
GRANT REPLICATION SLAVE  ON *.* TO 'repluser'@'HOST' IDENTIFIED BY 'replpass';
```

查看从二进制日志的文件和位置开始进行复制

```mysql
SHOW MASTER LOG;
```

**从节点配置：**

启动中继日志

```mysql
[mysqld]
server_id=# #为当前节点设置一个全局惟的ID号
log-bin
read_only=ON #设置数据库只读，针对supper user无效
relay_log=relay-log #relay log的文件路径，默认值hostname-relay-bin
relay_log_index=relay-log.index  #默认值hostname-relay-bin.index
```

使用有复制权限的用户账号连接至主服务器，并启动复制线程

```mysql
#8.0.23之前语法
CHANGE MASTER TO MASTER_HOST='masterhost',
MASTER_USER='repluser',
MASTER_PASSWORD='replpass',
MASTER_LOG_FILE='mariadb-bin.xxxxxx',
MASTER_LOG_POS=#;

#8.0.23之后语法
CHANGE REPLICATION SOURCE TO

START SLAVE [IO_THREAD|SQL_THREAD];
SHOW SLAVE STATUS;
```

范例：新建主从复制

```mysql
配置主节点
#修改配置文件
[root@centos8 backup]# vim /etc/my.cnf.d/mysql-server.cnf
[mysqld]
server_id=2
log_bin=/data/mysql-bin

#创建复制用户
mysql> CREATE USER 'repl'@'10.0.0.%' IDENTIFIED BY 'passwd';
Query OK, 0 rows affected (0.04 sec)
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'10.0.0.%';
Query OK, 0 rows affected (0.01 sec)

#查看二进制日志位置
mysql> show master logs;
+------------------+-----------+-----------+
| Log_name         | File_size | Encrypted |
+------------------+-----------+-----------+
| mysql-bin.000001 |       179 | No        |
| mysql-bin.000002 |       179 | No        |
| mysql-bin.000003 |       948 | No        |
| mysql-bin.000004 |       203 | No        |
| mysql-bin.000005 |       203 | No        |
| mysql-bin.000006 |       203 | No        |
| mysql-bin.000007 |      1626 | No        |
| mysql-bin.000008 |       687 | No        |
+------------------+-----------+-----------+

从节点配置
#修改配置文件，建议将二进制日志也开启
[root@centos8 ~]# vim /etc/my.cnf.d/mysql-server.cnf
[mysqld]
server-id=3
log_bin=/data/mysql-bin

#启动数据库服务
[root@centos8 ~]# systemctl start mysqld

#添加复制信息
[root@centos8 ~]# mysql
mysql> help change master to
mysql> CHANGE MASTER TO
    ->   MASTER_HOST='10.0.0.2',
    ->   MASTER_USER='repl',
    ->   MASTER_PASSWORD='passwd',
    ->   MASTER_PORT=3306,
    ->   MASTER_LOG_FILE='mysql-bin.000008',
    ->   MASTER_LOG_POS=687,
    ->   MASTER_CONNECT_RETRY=10;
Query OK, 0 rows affected, 2 warnings (0.11 sec)
mysql>  start slave;
Query OK, 0 rows affected (0.02 sec)

mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 10.0.0.2
                  Master_User: repl
        			Seconds_Behind_Master: 0	##复制的延迟时间

测试
#主服务器创建数据库
mysql> create database db2;
Query OK, 1 row affected (0.04 sec)

#查看从服务器是否同步
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| db2                |
```

#### 6.1.3 主从复制的注意事项

1. 限制从服务器为只读，否则会导致复制出错

```mysql
read_only=ON
#注意：此限制对拥有SUPER权限的用户均无效
```

2.清除从节点信息

```mysql
RESET SLAVE #从服务器清除master.info ，relay-log.info, relay log ，开始新的relay log
RESET SLAVE  ALL #清除所有从服务器上设置的主服务器同步信息，如HOST，PORT, USER和
PASSWORD 等
```

3.复制错误的解决方法

```mysql
#系统变量，指定跳过复制事件的个数，该变量适用于下 START REPLICA | SLAVE一条语句；下 START REPLICA | SLAVE一条语句还将该值更改回 0
SET GLOBAL sql_slave_skip_counter = N

#服务器选项，只读系统变量，指定跳过事件的ID
[mysqld]
slave_skip_errors=1007|ALL
```

4.START SLAVE 语句，指定执到特定的点

```mysql
START SLAVE [thread_types]
START SLAVE [SQL_THREAD] UNTIL
   MASTER_LOG_FILE = 'log_name', MASTER_LOG_POS = log_pos
START SLAVE [SQL_THREAD] UNTIL
   RELAY_LOG_FILE = 'log_name', RELAY_LOG_POS = log_pos
thread_types:
   [thread_type [, thread_type] ... ]
thread_type: IO_THREAD | SQL_THREAD
```

### 6.2 MySQL 中间件代理服务器

简单来说中间件代理服务器是为了实现数据库的分表分库，将数据库拆分开来以应对海量的数据存储，更多详细的介绍可以参看Mycat的官方文档，文档中对于中间件代理给出了很详细的解释http://www.mycat.org.cn/document/mycat-definitive-guide.pdf

#### 6.2.1 Mycat实战

下面将通过一个范例实现Mycat中间件代理

**系统环境**

```mysql
#主机系统版本
[root@centos8 ~]# cat /etc/centos-release
CentOS Linux release 8.2.2004 (Core) 

#主机分配
mycat-server 10.0.0.5 内存建议2G以上
mysql-master 10.0.0.2 #负责数据写入
mysql-slave 10.0.0.3	#负责数据读取

#关闭防火墙和selinux
systemctl stop firewalld
setenforce 0
```

**1、创建主从数据库**

这一步在之前主从复制案例中已经实现，此处不再重新部署。

**2、在10.0.0.5安装mycat并启动**

```mysql
#下载并安装JDK
[root@centos8 ~]# yum -y install java mysql
[root@centos8 ~]# java -version
openjdk version "1.8.0_292"
OpenJDK Runtime Environment (build 1.8.0_292-b10)
OpenJDK 64-Bit Server VM (build 25.292-b10, mixed mode)

#下载安装mycat
http://dl.mycat.org.cn/1.6.7.6/20210303094759/
[root@centos8 ~]# ls
anaconda-ks.cfg  jdk-16.0.1_linux-x64_bin.rpm  Mycat-server-1.6.7.6-release-20210303094759-linux.tar.gz
[root@centos8 ~]# mkdir /app
[root@centos8 ~]# tar xvf Mycat-server-1.6.7.6-release-20210303094759-linux.tar.gz -C /app/

#配置环境变量
[root@centos8 ~]# echo 'PATH=/app/mycat/bin:$PATH' > /etc/profile.d/mycat.sh
[root@centos8 ~]# source /etc/profile.d/mycat.sh 

#查看端口情况
[root@centos8 ~]# ss -ntl
State         Recv-Q        Send-Q               Local Address:Port               Peer Address:Port        
LISTEN        0             128                        0.0.0.0:22                      0.0.0.0:*           
LISTEN        0             128                           [::]:22                         [::]:*   

#启动mycat
[root@centos8 ~]# mycat start
Starting Mycat-server...
[root@centos8 ~]# ss -tpln
State    Recv-Q   Send-Q     Local Address:Port      Peer Address:Port                                     
LISTEN   0        1              127.0.0.1:32000          0.0.0.0:*      users:(("java",pid=15136,fd=4))   
LISTEN   0        128              0.0.0.0:22             0.0.0.0:*      users:(("sshd",pid=991,fd=4))     
LISTEN   0        50                     *:1984                 *:*      users:(("java",pid=15136,fd=73))  
LISTEN   0        128                    *:8066                 *:*      users:(("java",pid=15136,fd=97))  
LISTEN   0        50                     *:43973                *:*      users:(("java",pid=15136,fd=72))  
LISTEN   0        128                    *:9066                 *:*      users:(("java",pid=15136,fd=93))  
LISTEN   0        128                 [::]:22                [::]:*      users:(("sshd",pid=991,fd=6))     
LISTEN   0        50                     *:43385                *:*      users:(("java",pid=15136,fd=74))

#查看日志是否启动成功
[root@centos8 ~]# tail /app/mycat/logs/wrapper.log
INFO   | jvm 5    | 2021/06/27 04:38:07 | Error: A fatal exception has occurred. Program will exit.
FATAL  | wrapper  | 2021/06/27 04:38:07 | There were 5 failed launches in a row, each lasting less than 300 seconds.  Giving up.
FATAL  | wrapper  | 2021/06/27 04:38:07 |   There may be a configuration problem: please check the logs.
STATUS | wrapper  | 2021/06/27 04:38:08 | <-- Wrapper Stopped
STATUS | wrapper  | 2021/06/27 04:55:51 | --> Wrapper Started as Daemon
STATUS | wrapper  | 2021/06/27 04:55:51 | Launching a JVM...
INFO   | jvm 1    | 2021/06/27 04:55:55 | Wrapper (Version 3.2.3) http://wrapper.tanukisoftware.org
INFO   | jvm 1    | 2021/06/27 04:55:55 |   Copyright 1999-2006 Tanuki Software, Inc.  All Rights Reserved.
INFO   | jvm 1    | 2021/06/27 04:55:55 | 
INFO   | jvm 1    | 2021/06/27 04:55:59 | MyCAT Server startup successfully. see logs in logs/mycat.log
```

**3、修改schema.xml实现读写分离策略**

```mysql
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">
<schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100" dataNode ="dn1"></schema>
        <dataNode name="dn1" dataHost="localhost1" database="mycat" />
        <dataHost name="localhost1" maxCon="1000" minCon="10" balance="3" writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
                <heartbeat>select user()</heartbeat>
                <writeHost host="host1" url="10.0.0.2:3306" user="root" password="123456">
                <readHost host="host2" url="10.0.0.3:3306" user="root" password="123456"/>
                </writeHost>
        </dataHost>
</mycat:schema>

#重启mycat
[root@centos8 conf]# mycat restart

#遇见的报错
schema TESTDB didn't config tables,so you must set dataNode property
schema标签中属性与可嵌套的table 标签有依赖关系 。如果不设置table标签，就必须设置schema标签中dataNode属性。
```

**4、在后端主服务器创建用户并对mycat授权**

```mysql
mysql>  create database mycat;
Query OK, 1 row affected (0.06 sec)

mysql> CREATE USER 'root'@'10.0.0.5' IDENTIFIED BY '123456';
Query OK, 0 rows affected (0.03 sec)

mysql> GRANT ALL ON *.* TO 'root'@'10.0.0.5';
Query OK, 0 rows affected (0.05 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)

#若mycat使用账号对后端服务器无权限会报下面错误
mysql> show tables;
ERROR 1184 (HY000): Invalid DataSource:0
```

**5、在Mycat服务器上连接并测试**

```mysql
[root@centos8 conf]# mysql -uroot -p123456 -h127.0.0.1 -P8066 -DTESTDB
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.

mysql>  create table t1(id int);
Query OK, 0 rows affected (0.24 sec)
```

**6、通过通用日志确认实现读写分离**

```mysql
#开启通用日志功能
mysql> set global general_log=on;
Query OK, 0 rows affected (0.00 sec)

mysql> show variables like 'general_log';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| general_log   | ON    |
+---------------+-------+
1 row in set (0.01 sec)

#查看日志文件保存位置
mysql> show variables like 'general_log_file';
+------------------+----------------------------+
| Variable_name    | Value                      |
+------------------+----------------------------+
| general_log_file | /var/lib/mysql/centos8.log |
+------------------+----------------------------+
1 row in set (0.01 sec)

#设置日志文件保存位置
mysql> set global general_log_file='tmp/general.log';

#查看通用日志
[root@centos8 ~]# tail -f /var/lib/mysql/centos8.log
/usr/libexec/mysqld, Version: 8.0.21 (Source distribution). started with:
Tcp port: 3306  Unix socket: /var/lib/mysql/mysql.sock
Time                 Id Command    Argument
2021-06-27T09:38:27.480755Z	   16 Query	show variables like 'general_log'
2021-06-27T09:38:32.833139Z	  297 Query	select user()

#在mycat服务器执行查询命令
mysql> select * from tb2;

#在主从服务器上分别确认通用日志只有10.0.0.3服务器出现查询日志，表示已实现读写分离
2021-06-27T09:42:01.264139Z	  301 Query	select * from tb1
```

**7、停止从节点，MyCAT自动调度读请求至主节点**

```mysql
[root@centos8 ~]# systemctl stop mysqld

mysql>  select @@server_id;
+-------------+
| @@server_id |
+-------------+
|           2 |
+-------------+
1 row in set (0.01 sec)
```

注意：停止主节点，MyCAT不会自动调度读请求至从节点

# 9.自动运维工具ansible

## 1 Ansible 介绍和架构

### 1.1 Ansible介绍

ansible 的名称来自科幻小说《安德的游戏》中跨越时空的即时通信工具，使用它可以在相距数光年的 距离，远程实时控制前线的舰队战斗。在计算机中ansible则作为一个运维工具，可以实现批量管理主机、使用模块化方式自动安装复杂软件如K8s等功能。与ansible类似还有以下工具：

- Saltstack：python，一般需部署agent，执行效率更高
- Puppet：ruby, 功能强大，配置复杂，重型，适合大型环境
- Fabric：python，agentless
- Chef：ruby，国内应用少
- Cfengine
- func

**ansible工作方式**

在实现主机间基于key验证之后，ansible执行playbook时会自动连接至被管理主机，并将ansible模块推送到被管理主机上，之后ansible会在被管理主机上执行这些模块，并在执行完毕之后移除这些模块。

官方文档：https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

### 1.2 Ansible 特性

- 模块化：调用特定的模块完成特定任务，支持自定义模块，可使用任何编程语言写模块
- Paramiko（python对ssh的实现），PyYAML，Jinja2（模板语言）三个关键模块
- 基于Python语言实现
- 部署简单，基于python和SSH(默认已安装)，agentless，无需代理不依赖PKI（无需ssl）
- 安全，基于OpenSSH
- 幂等性：一个任务执行1遍和执行n遍效果一样，不因重复执行带来意外情况
- 支持playbook编排任务，YAML格式，编排任务，支持丰富的数据结构
- 较强大的多层解决方案 role

### 1.3 Ansible相关概念介绍

**控制节点（Control node）**

安装了Ansible的主机，调用命令来管理需要被管理的主机。控制节点不能安装在Windows上

**被管理节点（Managed nodes）**

使用Ansible来管理的设备，被管理节点无需安装Ansible

**清单（Inventory）**

被管理节点的列表

**模块（Modules）**

用于实现Ansible功能，类似于命令行中的各种命令，如ls等

**任务（TASK）**

用于PLAYBOOK中，指定要执行的模块

**剧本（PLAYBOOK）**

包含了多个顺序执行的TASK，还可以在其中定义变量等

## 2 Ansible 安装和入门

### 2.1 Ansible安装

ansible的安装方法有多种

#### 2.1.1 EPEL源的安装方式

```bash
#推荐使用该方式安装
[root@ansible ~]#yum install ansible
```

#### 2.1.2 pip安装

```bash
#pip默认安装最新版本
yum install python-pip python-devel
yum install gcc glibc-devel zibl-devel rpm-bulid openssl-devel
/usr/bin/python -m pip install --upgrade pip
pip install ansible --upgrade
```

#### 2.1.3 git安装

```bash
git clone git://github.com/ansible/ansible.git --recursive
cd ./ansible
source ./hacking/env-setup
```

还有很多其他安装方式，详细可以参考官方文档：https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

#### 2.1.4 确认ansible版本

```bash
[root@server1 ~]# ansible --version
ansible 2.9.21
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Nov 16 2020, 22:23:17) [GCC 4.8.5 20150623 (Red Hat 4.8.5-44)]
```

#### 2.1.5 windows主机配置

参考文档：https://docs.ansible.com/ansible/latest/user_guide/windows_setup.html

```bash
管理节点
#安装pywinrm用于连接被管理主机
[root@centos8 Python-3.8.11]# pip3.6 install -i https://pypi.tuna.tsinghua.edu.cn/simple pywinrm
#配置主机列表
[root@centos8 Python-3.8.11]# cat /etc/ansible/hosts 
[windows]
172.16.60.90
[windows:vars]
ansible_user="administratorHsykt"
ansible_password="Hlhthsykt@60.90"
ansible_connection="winrm"
ansible_winrm_transport="basic"
ansible_port=5985
ansible_winrm_scheme=http
ansible_winrm_server_cert_validation=ignore

被管理节点
Windows机器上的powershell版本5.0，Framework4.0
#查看winrm服务状态
winrm enumerate winrm/config/listener
#配置winrm服务
winrm quickconfig
winrm set winrm/config/service/auth '@{Basic="true"}'
winrm set winrm/config/service '@{AllowUnencrypted="true"}'

注意事项
1.powershell和Framework版本要满足官方要求
2.管理节点要安装pywinrm
```

### 2.2 Ansible主要文件

#### 2.2.1 配置文件

Ansible 的配置文件 /etc/ansible/ansible.cfg

```bash
[defaults]
#inventory     = /etc/ansible/hosts # 主机列表配置文件
#library = /usr/share/my_modules/ # 库文件存放目录
#remote_tmp = $HOME/.ansible/tmp #临时py命令文件存放在远程主机目录
#local_tmp     = $HOME/.ansible/tmp # 本机的临时命令执行目录
#forks         = 5   # 默认并发数
#sudo_user     = root # 默认sudo 用户
#ask_sudo_pass = True #每次执行ansible命令是否询问ssh密码
#ask_pass     = True  
#remote_port   = 22
#host_key_checking = False # 检查对应服务器的host_key，建议取消注释
#log_path=/var/log/ansible.log #日志文件，建议启用
#module_name = command   #默认模块，可以修改为shell模块
```

#### 2.2.2 Inventory设备清单文件

默认的inventory file为 /etc/ansible/hosts，主要用于指定要执行Ansible模块的主机

范例：

```bash
#如若目标主机使用了非默认的SSH端口，还可以在主机名称之后使用冒号加端口号来标明
[webservers]
www1.123.com:2222
www2.321.com

[websrvs]
www[1:100].example.com

[appsrvs]
10.0.0.[1:100]
```

#### 2.2.3 roles

默认位于/etc/ansible/roles/ ，主要用来存放角色

```bash
[root@server1 ~]# ll /etc/ansible/roles/ -d
drwxr-xr-x 2 root root 6 5月   5 04:39 /etc/ansible/roles/
```

### 2.3 Ansible的常用工具

- /usr/bin/ansible 主程序，临时命令执行工具
- /usr/bin/ansible-doc 查看配置文档，模块功能查看工具,相当于man
- /usr/bin/ansible-playbook 定制自动化任务，编排剧本工具,相当于脚本
- /usr/bin/ansible-pull 远程执行命令的工具
- /usr/bin/ansible-vault 文件加密工具
- /usr/bin/ansible-console 基于Console界面与用户交互的执行工具
- /usr/bin/ansible-galaxy 下载/上传优秀代码或Roles模块的官网平台

利用ansible实现管理的主要方式：

- Ad-Hoc 即利用ansible命令，主要用于临时命令使用场景
- Ansible-playbook 主要用于长期规划好的，大型项目的场景，需要有前期的规划过程

#### 2.3.1 ansible-doc

显示模块的帮助信息

格式：

```bash
ansible-doc [options] [module...]
-l, --list       #列出可用模块
-s, --snippet #显示指定模块的playbook片段
```

#### 2.3.2 ansible

针对设置的主机执行单个任务playbook

格式：

```bash
ansible <host-pattern> [-m module_name] [-a "<module options>"]
```

选项：

- --version #显示版本
- -m module #指定模块，默认为command
- -v #详细过程 –vv -vvv更详细
- --list-hosts #显示主机列表，可简写 --list
- -k, --ask-pass #提示输入ssh连接密码，默认Key验证
- -C, --check #检查，并不执行
- -T, --timeout=TIMEOUT #执行命令的超时时间，默认10s
- -u, --user=REMOTE_USER #执行远程执行的用户
- -b, --become #代替旧版的sudo 切换
- --become-user=USERNAME #指定sudo的runas用户，默认为root
- -K, --ask-become-pass #提示输入sudo时的口令

**pattern的用法**

表格中列出了常用的pattern

| Description            | Pattern(s)                   | Targets                                             |
| :--------------------- | :--------------------------- | :-------------------------------------------------- |
| All hosts              | all (or *)                   |                                                     |
| One host               | host1                        |                                                     |
| Multiple hosts         | host1:host2 (or host1,host2) |                                                     |
| One group              | webservers                   |                                                     |
| Multiple groups        | webservers:dbservers         | all hosts in webservers plus all hosts in dbservers |
| Excluding groups       | webservers:!atlanta          | all hosts in webservers except those in atlanta     |
| Intersection of groups | webservers:&staging          | any hosts in webservers that are also in staging    |

同时还支持通配符和正则表达式

```bash
#通配符
ansible "*" -m ping
ansible 192.168.1.* -m ping
ansible "srvs" -m ping

#正则表达式
ansible "websrvs:dbsrvs" –m ping
ansible "~(web|db).*\.magedu\.com" –m ping
```

这里值得说明的是执行完成后会有不同颜色的返回结果，下面简单进行一个说明

- 绿色：执行成功并且不需要做改变的操作
- 黄色：执行成功并且对目标主机做变更
- 红色：执行失败

在配置文件中可以进行自定义

```bash
[root@server1 ~]# grep -A 14 '\[colors\]' /etc/ansible/ansible.cfg
[colors]
#highlight = white
#verbose = blue
#warn = bright purple
#error = red
#debug = dark gray
#deprecate = purple
#skip = cyan
#unreachable = red
#ok = green
#changed = yellow
#diff_add = green
#diff_remove = red
#diff_lines = cyan
```

### 2.4 Ansible常用模块

Ansible虽然模块众多，但最常用的模块也就2，30个而已，针对特定业务只用10几个模块。

官方文档：https://docs.ansible.com/ansible/latest/collections/index.html

#### 2.4.1 Command 模块

功能：在远程主机执行命令，此为默认模块，所以可忽略-m选项

注意：variables like `$HOSTNAME` and operations like `"*"`, `"<"`, `">"`, `"|"`, `";"` and `"&"` will not work，所以一般会使用shell模块来代替Command 模块

范例：

```bash
[root@server1 ~]# ansible lvyou -a 'ls /data'
172.16.60.218 | CHANGED | rc=0 >>
123.txt
```

#### 2.4.2 shell 模块

功能：和command相似，用shell执行命令（推荐）

范例：

```bash
[root@centos8 .ssh]# ansible innet -m shell -a 'ls /root'
172.16.222.101 | CHANGED | rc=0 >>
anaconda-ks.cfg
172.16.111.100 | CHANGED | rc=0 >>
anaconda-ks.cfg

[root@centos8 .ssh]# ansible innet -m shell -a 'echo $HOSTNAME'
172.16.222.101 | CHANGED | rc=0 >>
centos8
172.16.111.100 | CHANGED | rc=0 >>
centos8.1
```

将shell设置为默认模块

```bash
[root@ansible ~]#vim /etc/ansible/ansible.cfg
#修改下面一行
module_name = shell
```

#### 2.4.3 Script 模块

**功能：在远程主机上运行ansible服务器上的脚本(无需执行权限)**

范例：

```bash
ansible inint -m script -a /data/test.sh
```

#### 2.4.4 Copy 模块

**功能：从ansible服务器主控端复制文件到远程主机**

范例：

```bash
#如目标存在，默认覆盖，此处指定先备份，注意文件内容不一致时才会产生备份文件
[root@centos8 .ssh]# ansible innet -m copy -a "src=/root/123.txt dest=/data backup=yes"

#将指定内容，直接生成目标文件
[root@centos8 .ssh]# ansible innet -m copy -a "content='123\nab' dest=/data/test.txt"

#复制/etc目录自身,注意/etc/后面没有/，此时会将/etc整个目录拷贝过去，dest目录如果没有会自动创建
[root@centos8 .ssh]# ansible innet -m copy -a "src=/etc dest=/backup"
```

#### 2.4.5 File模块

**功能：创建文件指定文件属性**

范例：

```bash
#创建空文件
[root@centos8 .ssh]# ansible innet -m file  -a 'path=/data/test1.txt state=touch'

#删除文件
[root@centos8 .ssh]# ansible innet -m file  -a 'path=/data/test1.txt state=absent'

#创建目录
ansible all -m file -a "path=/data/mysql state=directory owner=mysql group=mysql"

#创建软链接
ansible all -m file -a 'src=/data/testfile dest=/data/testfile-link state=link'
```

#### 2.4.6 unarchive 模块

**功能：解包解压缩**

常见参数：

- copy：默认为yes，当copy=yes，拷贝的文件是从ansible主机复制到远程主机上，如果设置为 copy=no，会在远程主机上寻找src源文件
- remote_src：和copy功能一样且互斥，yes表示在远程主机，不在ansible主机，no表示文件在 ansible主机上
- src：源路径，可以是ansible主机上的路径，也可以是远程主机(被管理端或者第三方主机)上的路 径，如果是远程主机上的路径，则需要设置copy=no
- dest：远程主机上的目标路径
- mode：设置解压缩后的文件权限

范例：

```bash
ansible all -m unarchive -a 'src=/data/foo.tgz dest=/var/lib/foo'
ansible all -m unarchive -a 'src=https://example.com/example.zip dest=/data copy=no'
```

#### 2.4.7 Archive 模块

**功能：管理主机名**

范例：

```bash
ansible websrvs -m archive  -a 'path=/var/log/ dest=/data/log.tar.bz2 format=bz2'
```

#### 2.4.8 Cron 模块

**功能：实现计划任务**

范例：

```bash
#创建计划任务
ansible 10.0.0.8 -m cron -a 'hour=2 minute=30 weekday=1-5 name="backup mysql"
job=/root/mysql_backup.sh'

#禁用计划任务
ansible websrvs   -m cron -a "minute=*/5 job='/usr/sbin/ntpdate 172.20.0.1
&>/dev/null' name=Synctime disabled=yes"

#启用计划任务
ansible websrvs   -m cron -a "minute=*/5 job='/usr/sbin/ntpdate 172.20.0.1
&>/dev/null' name=Synctime disabled=no"

#删除计划任务
ansible websrvs -m cron -a "name='backup mysql' state=absent"
ansible websrvs -m cron -a 'state=absent name=Synctime'
```

#### 2.4.9 Yum 模块

**功能：管理软件包，只支持RHEL，CentOS，fedora，不支持Ubuntu其它版本**

范例：

```bash
ansible innet -m yum -a 'name=httpd state=present'  #安装
ansible innet -m yum -a 'name=httpd state=absent'   #删除
```

#### 2.4.10 Setup 模块

**功能： setup 模块来收集主机的系统信息，这些 facts 信息可以直接以变量的形式使用，但是如果主机 较多，会影响执行速度，可以使用 gather_facts: no 来禁止 Ansible 收集 facts 信息**

范例：

```bash
#收集全部信息
[root@centos8 .ssh]# ansible innet -m setup

#收集指定主机信息
[root@centos8 .ssh]# ansible innet -m setup -a 'filter=ansible_python_version'
```

## 3 Playbook

### 3.1 Plyabook介绍

playbook 剧本是由一个或多个"play"组成

play的主要功能在于将预定义的一组主机，装扮成事先通过ansible中的task定义好的角色。Task实际是 调用ansible的一个module，将多个play组织在一个playbook中，即可以让它们联合起来，按事先编排的机制执行预定义的动作，从而完成复杂的主机管理功能

### 3.2 YAML

#### 3.2.1 YAML简介

由于Playbook 文件是采用YAML语言编写的，所以下面对YAML进行简单的介绍

YAML是一种和XML、JSON类似的语言，JSON主要用于进行网络中数据交换，YAML则通常用来配置。XML既可以用于数据交换也能配置，不过语法比前两种繁琐。

**可以用工具互相转换**

https://www.json2yaml.com/

http://www.bejson.com/json/json2yaml/

#### 3.2.2 YAML特点

- 易读性较强
- 交互性较好
- 易于实现
- 表达能力强，扩展好
- 有一个一致的信息模型

#### 3.2.3 YAML语法简介

- 在单一文件第一行，用连续三个连字号"-" 开始（也可以省略），还有选择性的连续三个点号( ... )用来表示文件的结尾
- 次行开始正常写Playbook的内容，一般建议写明该Playbook的功能
- 使用#号注释代码
- 缩进必须统一，TAB和空格不能混用
- 缩进的级别也必须一致
- key/value可同行也可换行写，同行使用，进行分隔
- 一般YAML文件扩展名为yml或yaml

**List列表**

列表由多个元素组成，每个元素放在不同行，且元素前均使用"-"打头，并且 - 后有一个空格, 或者将所 有元素用 [ ] 括起来放在同一行

范例：

```bash
#不同行,行以-开头,后面有一个空格
# A list of tasty fruits
- Apple
- Orange
- Strawberry
- Mango

#同一行
[Apple,Orange,Strawberry,Mango]
```

**Dictionary字典**

字典由多个key与value构成，key和value之间用 ：分隔, 并且 : 后面有一个空格，所有k/v可以放在一 行，或者每个 k/v 分别放在不同行

范例：

```bash
#不同行
# An employee record
name: Example Developer
job: Developer
skill: Elite

#同一行,也可以将ke
y:value放置于{}中进行表示，用,分隔多个key:value
# An employee record
{name: "Example Developer", job: "Developer", skill: "Elite"}
```

### 3.3 Playbook 核心组件

常见组件类型如下:

- Hosts 执行的远程主机列表
- Tasks 任务集,由多个task的元素组成的列表实现,每个task是一个字典
- Variables 内置变量或自定义变量在playbook中调用
- Templates 模板，可替换模板文件中的变量并实现一些简单逻辑的文件
- Handlers 和 notify 结合使用，由特定条件触发的操作，满足条件方才执行，否则不执行
- tags 标签 指定某条任务执行，用于选择运行playbook中的部分代码。ansible具有幂等性，因此 会自动跳过没有变化的部分，即便如此，有些代码为测试其确实没有发生变化的时间依然会非常地 长。此时，如果确信其没有变化，就可以通过tags跳过此些代码片断
- 一个完整的代码块功能需最少元素需包括 name 和 task,一个name只能包括一个task

更多组件和详细的组件说明可以参看官方文档：https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html

下面分别用shell和palybook实现httpd安装

范例：

```bash
#SHELL脚本实现
#!/bin/bash
# 安装Apache
yum install --quiet -y httpd
# 复制配置文件
cp /tmp/httpd.conf /etc/httpd/conf/httpd.conf
cp/tmp/vhosts.conf /etc/httpd/conf.d/
# 启动Apache，并设置开机启动
systemctl enable --now httpd

#Playbook实现
---
- hosts: websrvs
 remote_user: root
 tasks:
   - name: "安装Apache"
     yum: name=httpd
   - name: "复制配置文件"
     copy: src=/tmp/httpd.conf dest=/etc/httpd/conf/
   - name: "复制配置文件"
     copy: src=/tmp/vhosts.conf dest=/etc/httpd/conf.d/
   - name: "启动Apache，并设置开机启动"
     service: name=httpd state=started enabled=yes
```

### 3.4 playbook命令

格式：

```bash
ansible-playbook <filename.yml> ... [options]
```

常用选项：

- -C --check #只检测可能会发生的改变，但不真正执行操作
- --list-hosts #列出运行任务的主机
- --list-tags #列出tag
- --list-tasks #列出task
- --limit 主机列表 #只针对主机列表中的特定主机执行
- -v -vv -vvv #显示过程

范例：

```bash
[root@centos8 data]# cat hello.yml 
---
- hosts: innet
  tasks:
    - name: hello
      command: echo "hello ansible"
      
[root@centos8 data]# ansible-playbook -C hello.yml
[root@centos8 data]# ansible-playbook hello.yml
```

### 3.5 Playbook初步

#### 3.5.1 Playbook使用handlers和notify

Handlers本质是task list，其作用类似一个动作，当task执行完毕时使用notify触发才会执行。

范例：当配置文件发生改变，对服务进行重启操作

```bash
---
- hosts: websrvs
 remote_user: root
 gather_facts: no
 tasks:
    - name: Install httpd
     yum: name=httpd state=present
    - name: Install configure file
     copy: src=files/httpd.conf dest=/etc/httpd/conf/
     notify: restart httpd
    - name: ensure apache is running
      service: name=httpd state=started enabled=yes
  
 handlers:
    - name: restart httpd
      service: name=httpd state=restarted
```

#### 3.5.2 Playbook中使用tags组件

在playbook文件中，可以利用tags组件，为特定 task 指定标签，当在执行playbook时，可以只执行特 定tags的task,而非整个playbook文件

范例：

```bash
# tags example
- hosts: websrvs
 remote_user: root
 gather_facts: no
  
 tasks:
    - name: Install httpd
     yum: name=httpd state=present
    - name: Install configure file
     copy: src=files/httpd.conf dest=/etc/httpd/conf/
     tags: conf
    - name: start httpd service
     tags: service
      service: name=httpd state=started enabled=yes
      
[root@centos8 data]#ansible-playbook –t conf,service httpd.yml
```

### 3.6 Playbook中变量的使用

**变量的调用：**

通过{{ variable_name }} 调用变量，且变量名前后建议加空格，有时用"{{ variable_name }}"才生效

调用较为简单，这里主要介绍几种定义变量的方式

#### 3.6.1 使用setup模块中的变量

本模块自动在playbook调用，不要用ansible命令调用

范例：

```bash
[root@centos8 data]# cat var1.yml 
---
#var1.yml
- hosts: innet
  
  tasks:
    - name: create log file
      file: name=/data/{{ ansible_nodename }}.log state=touch 
```

#### 3.6.2 在命令行中定义变量

范例：

```bash
[root@centos8 data]# cat var2.yml 
---
- hosts: innet

  tasks:
    - name: touch file
      file: name=/data/{{ file }}.log state=touch

[root@centos8 data]# ansible-playbook -e file=test var2.yml
```

#### 3.6.3 在playbook文件中定义变量

范例：

```bash
[root@centos8 data]# cat va3.yml 
---
- hosts: innet
  remote_user: root
  vars:
    collect_info: "/data/{{ansible_nodename}}/"
  tasks:
    - name: create IP directory
      file: name="{{collect_info}}" state=directory
       
[root@centos8 data]# ansible-playbook -e file=test var3.yml
```

#### 3.6.4 使用变量文件

可以在一个独立的playbook文件中定义变量，在另一个playbook文件中引用变量文件中的变量，比 playbook中定义的变量优化级高

范例：

```bash
vim vars.yml
---
# variables file
package_name: mariadb-server
service_name: mariadb
vim var5.yml
---
#install package and start service
- hosts: dbsrvs
 remote_user: root
 vars_files:
    - vars.yml
 tasks:
    - name: install package
     yum: name={{ package_name }}
     tags: install
    - name: start service
      service: name={{ service_name }} state=started enabled=yes
```

#### 3.6.5 主机清单文件中定义变量

**主机变量**

为指定的主机定义变量

范例：

```bash
[websrvs]
www1.hello.com http_port=80 maxRequestsPerChild=808
www2.hello.com http_port=8080 maxRequestsPerChild=909
```

**组（公共）变量**

给指定组内所有主机上的在playbook中可用的变量，如果和主机变是 同名，优先级低于主机变量

范例：

```bash
[websrvs]
www1.hello.com http_port=8080
www2.hello.com
[websrvs:vars]
http_port=80
ntp_server=ntp.magedu.com
nfs_server=nfs.magedu.com
```

### 3.7 template 模板

模板是一个文本文件，可以做为生成文件的模版，并且模板文件中还可嵌套jinja语法

#### 3.7.1 jinja2语言

官方文档：https://jinja.palletsprojects.com/en/3.0.x/

中文版本：http://docs.jinkan.org/docs/jinja2/#

#### 3.7.2 template

官方介绍：https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html

范例：

```bash
使用 if条件判断，决定是否生成相关的配置信息
#yml文件
[root@centos8 data]# cat temp5.yml 
---
- hosts: innet
  vars:
    nginx_vhosts:
     - web1:
       listen: 8080
       root: "/var/www/nginx/web1/"
     - web2:
       listen: 8080
       server_name: "web2.magedu.com"
       root: "/var/www/nginx/web2/"
     - web3:
       listen: 8080
       server_name: "web3.magedu.com"
       root: "/var/www/nginx/web3/"
  tasks:
     - name: template config to
       template: src=nginx.conf5.j2 dest=/data/nginx5.conf #也可以写绝对路径，使用此处写法将模板文件放在同级目录也可以执行

#模板文件位置
[root@centos8 data]# cat ./templates/nginx.conf5.j2 
{% for vhost in nginx_vhosts %}
server {
   listen {{ vhost.listen }}
   {% if vhost.server_name is defined %}
server_name {{ vhost.server_name }}
   {% endif %}
root  {{ vhost.root }}
}
{% endfor %}

#生成的文件
[root@centos8 data]# cat nginx5.conf 
server {
   listen 8080
   root  /var/www/nginx/web1/
}
server {
   listen 8080
   server_name web2.magedu.com
   root  /var/www/nginx/web2/
}
server {
   listen 8080
   server_name web3.magedu.com
   root  /var/www/nginx/web3/
}
```

## 4 roles角色

roles就是通过分别将变量、文件、任务、模板及处理器放置于单独的目录中，并可以便 捷地include它们的一种机制。将任务拆分成多个模块，适用于较为复杂的场景，提高代码复用性。

官方文档：https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html

### 4.1 roles结构

roles目录结构如下

```
# playbooks
site.yml
webservers.yml
fooservers.yml
roles/
    common/
        tasks/
        handlers/
        library/
        files/
        templates/
        vars/
        defaults/
        meta/
    webservers/
        tasks/
        defaults/
        meta/
```

默认情况Ansible会去查找角色每个目录下的main.yml文件中内容

- `tasks/main.yml` - the main list of tasks that the role executes.
- `handlers/main.yml` - handlers, which may be used within or outside this role.
- `library/my_module.py` - modules, which may be used within this role
- `defaults/main.yml` - 设定默认变量时使用此目录中的main.yml文件，比vars的优先级低
- `vars/main.yml` - other variables for the role.
- `files/main.yml` - 存放由copy或script模块等调用的文件
- `templates/main.yml` - templates that the role deploys.
- `meta/main.yml` - metadata for the role, including role dependencies.

### 4.2 role使用

调用role步骤

```bash
1 创建以roles命名的目录
2 在roles目录中分别创建以各角色名称命名的目录，如webservers等
3 在每个角色命名的目录中分别创建files、handlers、meta、tasks、templates和vars目录；用不到
的目录可以创建为空目录，也可以不创建
4 在playbook文件中，调用各角色
```

**在playbook中调用role方法**

范例：

```bash
---
- hosts: websrvs
 remote_user: root
 roles:
 - mysql
 - memcached
 - nginx 
 
#键role用于指定角色名称，后续的k/v用于传递变量给角色
---
- hosts: all
 remote_user: root
 roles:
    - mysql
    - { role: nginx, username: nginx }
    
#还可基于条件测试实现角色调用
---
- hosts: all
 remote_user: root
 roles:
   - { role: nginx, username: nginx, when: ansible_distribution_major_version
== '7' }
```

**roles 中使用 tags**

```bash
#nginx-role.yml
---
- hosts: websrvs
 remote_user: root
 roles:
    - { role: nginx ,tags: [ 'nginx', 'web' ] ,when:
ansible_distribution_major_version == "6" }
    - { role: httpd ,tags: [ 'httpd', 'web' ] }
    - { role: mysql ,tags: [ 'mysql', 'db' ] }
    - { role: mariadb ,tags: [ 'mariadb', 'db' ] }
ansible-playbook --tags="nginx,httpd,mysql" nginx-role.yml
```

### 4.3 实战案例

#### 4.3.1 roles实现zabbix-agent批量部署

```bash
[root@centos8 data]# tree roles/
roles/
└── zabbix
    ├── tasks
    │   ├── main.yml
    │   └── zabbixconf.yml
    └── templates
        └── zabbix.conf.j2

[root@centos8 data]# cat roles/zabbix/tasks/main.yml 
- include: zabbixconf.yml
[root@centos8 data]# cat roles/zabbix/tasks/zabbixconf.yml 
---
- name: zabbix host set
  template: src=zabbix.conf.j2 dest=/data/zabbix.conf

[root@centos8 data]# cat roles/zabbix/templates/zabbix.conf.j2 
hostname {{ ansible_nodename }};
```

## 5 ansible相关链接

ansible模块共享社区：https://galaxy.ansible.com/home

# 10.LANMP架构

## 1 rpm实现LAMP

安装环境

```bash
10.0.0.2	HTTP
10.0.0.5 数据库服务器
[root@centos8 ~]# cat /etc/redhat-release 
CentOS Linux release 8.2.2004 (Core) 
```

### 1.1 RPM包的安装部署

范例：CentOS 8 默认使用factcgi模式，可以按下面步骤修改为httpd的模块方式

```bash
10.0.0.5安装数据库
#安装数据库
[root@centos8 ~]# yum -y install mariadb-server
[root@centos8 ~]# systemctl start mariadb
#授权远程连接账户
MariaDB [(none)]> grant all on *.* to 'test'@'10.0.0.%' identified by'123321';

在10.0.0.2上安装HTTP和PHP
#安装HTTP、PHP和数据库连接驱动
[root@centos8 ~]# yum -y install httpd php php-mysqlnd

[root@centos8 ~]# cat /etc/httpd/conf.modules.d/15-php.conf
<IfModule !mod_php5.c>
  <IfModule prefork.c>
    LoadModule php7_module modules/libphp7.so
  </IfModule>
</IfModule>
#从配置文件中看出模块支持prefork模式，所以将模块修改为支持httpd 模块prefork模式
[root@centos8 ~]# vim /etc/httpd/conf.modules.d/00-mpm.conf
LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
#LoadModule mpm_worker_module modules/mod_mpm_worker.so
#LoadModule mpm_event_module modules/mod_mpm_event.so

#修改为httpd模块方式，非必改项目
[root@centos8 html]#vim /etc/httpd/conf.d/php.conf
#<IfModule !mod_php5.c>
#  <IfModule !mod_php7.c>
#    # Enable http authorization headers
#    SetEnvIfNoCase ^Authorization$ "(.+)" HTTP_AUTHORIZATION=$1
#
#    <FilesMatch \.(php|phar)$>
#        SetHandler "proxy:unix:/run/php-fpm/www.sock|fcgi://localhost"
#    </FilesMatch>
#  </IfModule>
#</IfModule>

#添加测试页面
[root@centos8 ~]#cat /var/www/html/lamp.php
<?php
try {
        $user='test';		#数据库用户
        $pass='123321';		#数据库用户密码
        $dbh = new PDO('mysql:host=10.0.0.5;port=3306;dbname=mysql', $user, $pass);
        foreach($dbh->query('SELECT user,host from user') as $row) {
                print_r($row);
        }
        $dbh = null;
} catch (PDOException $e) {
        print "Error!: " . $e->getMessage() . "<br/>";
        die();
}
?>

#配置HTTP能识别.php文件
[root@centos8 ~]#vim /etc/httpd24/conf/httpd.conf
AddType application/x-httpd-php .php

#启动HTTP服务
[root@centos8 ~]#systemctl start httpd

#测试页面是否能访问
[root@centos8 ~]# curl http://10.0.0.2/lamp.php
Array
(
    [user] => test
    [0] => test
    [host] => 10.0.0.%
    [1] => 10.0.0.%
)
```

## 2 编译安装基于 FASTCGI 模式的wordpress

### 2.1 实验环境准备

```bash
10.0.0.2	HTTP+php(fastcgi模式)
10.0.0.5 mariadb数据库服务器

#软件版本
CentOS 8.2
mariadb 10.3.28
```

### 2.2 二进制安装Mariadb

在官方网站下载二进制安装包：https://downloads.mariadb.org/mariadb/10.3.30/

这里下载的是`mariadb-10.3.30-linux-systemd-x86_64.tar.gz (for systems with systemd)`版本，要求`systemd`和 GLIBC 2.19或者更高版本

```bash
#解压软件包并创建软链接
[root@centos8 src]# pwd
/usr/local/src
[root@centos8 src]# tar xf mariadb-10.3.30-linux-systemd-x86_64.tar.gz

#创建软链接方便之后版本升级，建议链接到/usr/local目录，这是mariadb.service的默认查找目录
[root@centos8 local]# ln -sv /usr/local/src/mariadb-10.3.30-linux-systemd-x86_64 mysql
'mysql' -> '/usr/local/src/mariadb-10.3.30-linux-systemd-x86_64'

#创建mariadb用户
[root@centos8 src]# useradd -r -s /sbin/nologin mysql
[root@centos8 src]# cd mysql/

#创建相应文件夹，赋予对应权限
[root@centos8 mysql]# chown -R root.root ./*
[root@centos8 mysql]# mkdir /data/mysql -p
[root@centos8 mysql]# chown -R mysql.mysql /data/mysql/
[root@centos8 mysql]# mkdir /etc/mysql
cat /etc/mysql/my.cnf
[mysqld]
datadir =/data/mysql
skip_name_resolve = ON

#安装数据库
[root@centos8 mysql]# ./scripts/mysql_install_db  --user=mysql --datadir=/data/mysql
Installing MariaDB/MySQL system tables in '/data/mysql' ...
OK

#配置环境变量
[root@centos8 mysql]# vim /etc/profile.d/lamp.sh
PATH=/usr/local/src/mysql/bin/:$PATH
[root@centos8 mysql]# . /etc/profile.d/lamp.sh

#拷贝服务启动配置
[root@centos8 local]# cp support-files/systemd/mariadb.service /usr/lib/systemd/system/mariadb.service
[root@centos8 local]# systemctl start mariadb

#连接数据库时报错
[root@centos8 mysql]# mysql -uroot
mysql: error while loading shared libraries: libtinfo.so.5: cannot open shared object file: No such file or directory
#安装对应库重新进入即可
[root@centos8 mysql]# yum install libncurses*

#创建wordpress数据库和账户
[root@centos8 mysql]# mysql -uroot
MariaDB [mysql]> create database wordpress;
Query OK, 1 row affected (0.004 sec)
MariaDB [mysql]> grant all on wordpress.* to wpuser@'10.0.0.%' identified by "database";
Query OK, 0 rows affected (0.007 sec)
```

### 2.3 编译安装httpd

官网下载安装包：https://apr.apache.org/download.cgi

apr-1.7.0.tar.gz 、apr-util-1.6.1.tar.gz、httpd-2.4.48.tar.gz

```bash
#安装依赖包
[root@centos8 data]# yum install gcc pcre-devel openssl-devel expat-devel -y

#解压安装包
[root@centos8 data]# tar xvf httpd-2.4.48.tar.gz
[root@centos8 data]# tar xvf apr-1.7.0.tar.gz
[root@centos8 data]# tar xvf apr-util-1.6.1.tar.gz

#将三者一并编译并安装
[root@centos8 data]# mv apr-1.7.0/ httpd-2.4.48/srclib/apr
[root@centos8 data]# mv apr-util-1.6.1 httpd-2.4.48/srclib/apr-util
[root@centos8 data]# cd httpd-2.4.48
./configure \
--prefix=/apps/httpd24 \
--enable-so \
--enable-ssl \
--enable-cgi \
--enable-rewrite \
--with-zlib \
--with-pcre \
--with-included-apr \
--enable-modules=most \
--enable-mpms-shared=all \
--with-mpm=event
[root@centos8 httpd-2.4.48]# make -j 4 && make install

#配置环境变量
[root@centos8 httpd-2.4.48]# vim /etc/profile.d/lamp.sh
PATH=/apps/httpd24/bin:$PATH
[root@centos8 httpd-2.4.48]# . /etc/profile.d/lamp.sh 

#创建用户组
[root@centos8 data]# useradd -s /sbin/nologin -r apache
[root@centos8 data]# vim /apps/httpd24/conf/httpd.conf
user apache
group apache

#添加到开机自动启动服务
[root@centos8 data]# vim /usr/lib/systemd/system/httpd.service
[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target
Documentation=man:httpd(8)
Documentation=man:apachectl(8)
[Service]
Type=forking
#EnvironmentFile=/etc/sysconfig/httpd
ExecStart=/apps/httpd24/bin/apachectl start
#ExecStart=/apps/httpd24/bin/httpd $OPTIONS -k start
ExecReload=/apps/httpd24/bin/apachectl graceful
#ExecReload=/apps/httpd24/bin/httpd $OPTIONS -k graceful
ExecStop=/apps/httpd24/bin/apachectl stop
KillSignal=SIGCONT
PrivateTmp=true
[Install]
WantedBy=multi-user.target

#启动
[root@centos8 data]# systemctl enable --now apache
```

### 2.4 编译安装 fastcgi 方式的 php 7.4.7

官方站点下载安装包：https://www.php.net/downloads

```bash
#安装相关依赖包
[root@centos8 data]# yum -y install gcc libxml2-devel bzip2-devel libmcrypt-devel sqlite-devel oniguruma

#编译安装PHP
[root@centos8 php-7.4.7]# cd php-7.4.7
./configure \
--prefix=/apps/php74 \
--enable-mysqlnd \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--with-openssl   \
--with-zlib \
--with-config-file-path=/etc \
--with-config-file-scan-dir=/etc/php.d \
--enable-mbstring \
--enable-xml \
--enable-sockets \
--enable-fpm \
--enable-maintainer-zts \
--disable-fileinfo
#环境检查时报错
Package 'oniguruma', required by 'virtual:world', not found
#解决方式，安装后即可解决问题
[root@centos8 php-7.4.7]# dnf --enablerepo=PowerTools install oniguruma-devel
#正式开始编译安装
[root@centos8 php-7.4.7]# make -j 4 && make install

#配置环境变量
[root@centos8 php-7.4.7]# vim /etc/profile.d/lamp.sh
PATH=/apps/php74/bin:/apps/httpd24/bin:$PATH
 . /etc/profile.d/lamp.sh

#准备PHP相关文件
[root@centos8 php-7.4.7]# cp php.ini-production /etc/php.ini	拷贝配置文件
[root@centos8 php-7.4.7]# cp sapi/fpm/php-fpm.service /usr/lib/systemd/system	服务启动配置
[root@centos8 php74]# cd /apps/php74/etc/
[root@centos8 etc]# cp php-fpm.conf.default php-fpm.conf
[root@centos8 etc]# cd php-fpm.d/
[root@centos8 php-fpm.d]# cp www.conf.default www.conf
[root@centos8 php-fpm.d]# vim ./www.conf
user apache
group apache
ping.path = /ping		打开ping界面

#支持opcache加速
[root@centos8 etc]# vim /etc/php.ini
[opcache]
; Determines if Zend OPCache is enabled
opcache.enable=1
zend_extension=opcache.so

#启动服务php-fpm
[root@centos8 etc]# systemctl daemon-reload
[root@centos8 etc]# systemctl enable --now php-fpm.service
```

### 2.5 配置php-fpm

```bash
[root@centos8 etc]# vim /apps/httpd24/conf/httpd.conf
#取消下面两行的注释
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
#修改下面行
<IfModule dir_module>
DirectoryIndex index.php index.html
</IfModule>
#加下面三行
AddType application/x-httpd-php .php
#AddType application/x-httpd-php-source .phps
ProxyRequests Off

#使用虚拟主机方式部署博客
<virtualhost *:80>
servername blog.wordpress.org
documentroot /data/wordpress
<directory /data/wordpress>
require all granted
</directory>
ProxyPassMatch ^/(.*\.php)$ fcgi://127.0.0.1:9000/data/wordpress/$1
#实现status和ping页面
ProxyPassMatch ^/(fpm_status|ping)$ fcgi://127.0.0.1:9000/$1
CustomLog "logs/access_wordpress_log" common
</virtualhost>
```

### 2.6 部署wordpress

官网下载wordpress：https://cn.wordpress.org/download/#download-install

```bash
#解压
[root@centos8 src]# tar xvf wordpress-5.7.2-zh_CN.tar.gz
[root@centos8 src]# mv wordpress/ /data
[root@centos8 src]# setfacl -R -m u:apache:rwx /data/wordpress/

#测试访问是否成功
浏览器打开http://10.0.0.2/
出现wordpress引导页面表示安装成功
```

# 11.网络文件共享服务

## 1 FTP服务

### 1.1 FTP工作原理

服务器会开放两个端口分别用于发送命令和传输数据

从服务器角度分为下面两种模式：

- 主动模式：即服务器主动连接客户端

   命令通道：21/tcp端口

   数据通道：20/TCP

- 被动模式：客户端主动连接

   命令通道：21/tcp端口

   数据通道：随机port

FTP服务状态码：

```
1XX：信息 		  125：数据连接打开
2XX：成功类状态 	200：命令OK     230：登录成功
3XX：补充类       331：用户名OK
4XX：客户端错误 	425：不能打开数据连接
5XX：服务器错误 	530：不能登录
```

### 1.2 常见的FTP软件

**服务端软件**

vsftpd：Very Secure FTP Daemon，CentOS 默认FTP服务器，最新版本是vsftpd-3.0.4，在2021年5月更新。官方站点：https://security.appspot.com/vsftpd.html

Filezilla：服务端只支持Windows，官方站点https://filezilla-project.org/index.php

**客户端软件**

ftp，lftp，lftpget，wget，curl等

### 1.3 vsftpd

vsftpd支持很多的配置选项，其配置文件位于`/etc/vsftpd/vsftpd.conf`，使用`man 5 vsftpd.conf`可以查看支持的配置选项，也可以访问官方网站：https://security.appspot.com/vsftpd/vsftpd_conf.html。

用户和其共享目录

- 匿名用户（映射为系统用户ftp ）共享文件位置：/var/ftp
- 系统用户共享文件位置：用户家目录
- 虚拟用户共享文件位置：为其映射的系统用户的家目录

```bash
[root@centos8 ~]# vim /etc/vsftpd/vsftpd.conf
listen_port=2121 		#命令通道监听端口,默认值为21

匿名用户相关配置
anonymous_enable=YES 				#支持匿名用户，CentOS8 默认不允许匿名
no_anon_password=YES 				#匿名用户略过口令检查 , 默认NO
anon_upload_enable=YES 				#匿名上传，注意:文件系统权限
anon_mkdir_write_enable=YES 		#匿名建目录
anon_world_readable_only=NO 		#只能下载全部读的文件, 默认YES
anon_umask=0333 						#指定匿名上传文件的umask，默认077，注意：0333中的0不能省略
anon_other_write_enable=YES 		#可删除和修改上传的文件, ，默认NO
chown_uploads=YES        			#指定匿名用户上传文件默认的所有者和权限，默认NO
chown_username=wang					
chown_upload_mode=0644			

系统用户相关配置
local_enable=YES 						#是否允许linux用户登录
write_enable=YES 						#允许linux用户上传文件
local_umask=022 						#指定系统用户上传文件的默认权限对应umask
guest_enable=YES 						#所有系统用户都映射成guest用户
guest_username=ftp   				#配合上面选项才生效，指定guest用户
local_root=/ftproot 					#指定guest用户登录所在目录,但不影响匿名用户的登录目录
chroot_local_user=YES 				#禁锢系统用户，默认NO，即不禁锢
chroot_list_enable=YES     		#默认是NO
chroot_list_file=/etc/vsftpd/chroot_list   #默认值
当chroot_local_user=YES和chroot_list_enable=YES时，则chroot_list中用户不禁锢，即白名单
当chroot_local_user=NO和chroot_list_enable=YES时， 则chroot_list中用户禁锢，即黑名单

日志相关配置
#wu-ftp 日志：默认启用
xferlog_enable=YES 					#启用记录上传下载日志，此为默认值
xferlog_std_format=YES 				#使用wu-ftp日志格式，此为默认值
xferlog_file=/var/log/xferlog 	#可自动生成， 此为默认值
#vsftpd日志：默认不启用
dual_log_enable=YES 					#使用vsftpd日志格式，默认不启用
vsftpd_log_file=/var/log/vsftpd.log 	#可自动生成， 此为默认值

用户的登录控制
userlist_enable=YES   				#此为默认值
userlist_deny=YES(默认值) 			 #黑名单,不提示口令，NO为白名单
userlist_file=/etc/vsftpd/user_list  #此为默认值
```

### vsftpd 虚拟用户

所有虚拟用户会统一映射为一个指定的系统帐号：访问共享位置，即为此系统帐号的家目录。

范例：实现基于文件验证的vsftpd虚拟用户

```bash
创建虚拟用户数据
[root@centos8 vsftpd]# cat /etc/vsftpd/vuser.txt 
ftpuser
123321
ftpuser2
123321
ftpuser3
123321
#生成数据库
[root@centos8 vsftpd]# db_load -T -t hash -f vuser.txt vuser.db
#设置访问权限
[root@centos8 ~]#chmod 600 /etc/vsftpd/vusers.*

创建映射的系统用户
[root@centos8 ~]#useradd -d /data/ftproot -s /sbin/nologin -r vuser
[root@centos8 ~]#mkdir -pv /data/ftproot/upload
[root@centos8 ~]#setfacl -m u:vuser:rwx /data/ftproot/upload

配置pam文件
[root@centos8 ~]#cat /etc/pam.d/vsftpd.db
auth required pam_userdb.so db=/etc/vsftpd/vuser
account required pam_userdb.so db=/etc/vsftpd/vuser

修改vsftpd配置文件
[root@centos8 ~]#vim /etc/vsftpd/vsftpd.conf
guest_enable=YES				#启用虚拟用户
guest_username=vuser			#指定虚拟用户映射的系统用户
pam_service_name=vsftpd.db	#pam配置文件
user_config_dir=/etc/vsftpd/conf.d/	#指定各个用户配置文件存放的路径

针对ftpuser用户进行配置
[root@centos8 vsftpd]# cat conf.d/ftpuser 
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
local_root=/data/awk/
```

注意：/etc/pam.d/vsftpd配置会影响到ftp用户登录

```bash
[root@centos8 vsftpd]# cat /etc/pam.d/vsftpd
#%PAM-1.0
session    optional     pam_keyinit.so    force revoke
auth       required	pam_listfile.so item=user sense=deny file=/etc/vsftpd/ftpusers onerr=succeed
auth       required	pam_shells.so		#仅允许用户的shell为 /etc/shells类型才能登录
auth       include	password-auth
account    include	password-auth
session    required     pam_loginuid.so
session    include	password-auth

上实验中将用户默认shell指定为/sbin/nologin，而/etc/shells默认无此类型，服务和用户都正常情况登录出现530情况，之后将/sbin/nologin添加至/etc/shells中，可以正常登录
```

## 2 NFS服务

### 2.1 NFS服务概述

NFS：Network File System 网络文件系统，基于内核的文件系统。Sun 公司开发，通过使用 NFS，用 户和程序可以像访问本地文件一样访问远端系统上的文件，基于RPC（Remote Procedure Call Protocol 远程过程调用）实现。

RPC是一个计算机通信协议，该协议允许运行于一台计算机的程序调用另一个地址空间的子进程，而程序员就像调用本地程序一样，无需额外地为这个交互作用编程（无需关注细节）。

[![NFS](https://www.cnblogs.com/bestvae/p/21.%E7%BD%91%E7%BB%9C%E6%96%87%E4%BB%B6%E5%85%B1%E4%BA%AB%E6%9C%8D%E5%8A%A1.assets/NFS.gif)](https://www.cnblogs.com/bestvae/p/21.网络文件共享服务.assets/NFS.gif)

### 2.2 NFS配置介绍

红帽文档地址：https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/managing_file_systems/exporting-nfs-shares_managing-file-systems#introduction-to-nfs_exporting-nfs-shares

### 2.3 实战案例

#### 2.3.1 实现主机上/var/www目录的共享

**实验环境准备**

```bash
10.0.0.2 服务端
10.0.0.3 客户端
```

**步骤**

```bash
10.0.0.2 服务端配置
#安装软件包
[root@centos8 vsftpd]# yum -y install nfs-utils

#准备共享目录
[root@centos8 vsftpd]# ll /var/www/
total 0
drwxr-xr-x. 2 root root  6 Nov  4  2020 cgi-bin
drwxr-xr-x. 4 root root 30 Apr 30 02:58 html

#编辑配置文件，将/var/www/共享给所有主机，给予读写权限
[root@centos8 vsftpd]# cat /etc/exports
/var/www *(rw)

#在不重启服务情况使配置文件生效
[root@centos8 vsftpd]# exportfs -r

10.0.0.3 客户端实现autofs
#安装软件包
[root@centos8 vsftpd]# yum -y install autofs

#检查本机挂载目录是否存在，以确保不会覆盖已存在目录
[root@centos8 vsftpd]# ll /var/www
ls: cannot access '/var/www': No such file or directory

#相对路径配置挂载目录
[root@centos8 vsftpd]# vim /etc/auto.master
/opt /etc/auto.opt
[root@centos8 www]# cat /etc/auto.opt 
www    -fstype=nfs  10.0.0.2:/var/www/

#重启autofs服务
[root@centos8 /]# systemctl restart autofs

#进入目录查看挂载文件
[root@centos8 /]# cd /opt/www/
注意这里直接进入/opt目录是不能看见www文件夹的，需要先进入www文件夹才会自动挂载上
```

## 3 samba服务

### 3.1 服务简介

Samba软件和NFS服务软件主要功能都是以挂载方式实现文件的共享，不过Samba更加适用于Windows和Linux主机间的文件共享，而一般Linux主机间的文件共享NFS服务更为常用。

Samba软件具有很多功能，这里主要介绍共享文件服务

官方文档：https://wiki.samba.org/index.php/Samba_File_Serving

### 3.2 SAMBA软件介绍

相关包：

- samba 提供smb服务器端
- samba-client 客户端软件
- samba-common 通用软件
- cifs-utils smb客户端工具
- samba-winbind 和AD相关

相关服务进程：

- smbd 提供smb（cifs）服务 TCP:139,445

配置文件：

- /etc/samba/smb.conf 帮助参看：man smb.conf

客户端工具：

- smbclient，mount.cifs

### 3.3 实战案例

实现不同samba用户访问不同的目录

```bash
#在服务器上安装samba包
[root@centos8 ~]# yum -y install samba

#创建登录用户samba1、samba2、samba3并指定密码为passwd
[root@centos8 ~]# useradd -M -s /sbin/nologin smb1
[root@centos8 ~]# useradd -M -s /sbin/nologin smb2
[root@centos8 ~]# useradd -M -s /sbin/nologin smb3
[root@centos8 ~]# echo 'passwd' | passwd --stdin smb1
Changing password for user smb1.
passwd: all authentication tokens updated successfully.
[root@centos8 ~]# echo 'passwd' | passwd --stdin smb2
Changing password for user smb2.
passwd: all authentication tokens updated successfully.
[root@centos8 ~]# echo 'passwd' | passwd --stdin smb3
Changing password for user smb3.
passwd: all authentication tokens updated successfully.

#将用户添加到samba数据库
[root@centos8 ~]# smbpasswd -a smb2
New SMB password:
Retype new SMB password:
Added user smb2.
[root@centos8 ~]# smbpasswd -a smb1
New SMB password:
Retype new SMB password:
Added user smb1.
[root@centos8 ~]# smbpasswd -a smb3
New SMB password:
Retype new SMB password:
Added user smb3.

#修改主配置文件
[root@centos8 ~]# vim /etc/samba/smb.conf
config file= /etc/samba/conf.d/%U
[share]
	Path=/data/dir
	Read only= NO
	Guest ok = NO

#添加用户配置文件
[root@centos8 dir1]# cat /etc/samba/conf.d/smb1 
[share]
Path=/data/dir1
Read only= NO 
Create mask=0644
[root@centos8 dir1]# cat /etc/samba/conf.d/smb2
[share]
path=/data/dir2

#创建文件夹
[root@centos8 samba]# mkdir /data/{dir,dir1,dir2}

#使用客户端测试访问
[root@centos8 ~]# smbclient //10.0.0.2/share -U smb1%passwd
Try "help" to get a list of possible commands.
smb: \> 
```

注意：如果要通过Windows访问需开启samba服务，不过samba服务不建议在Windows中开启,因为Windows1.1samba存在已知漏洞，有安全风险。

## 4 数据实时同步

### 4.1 实时同步介绍

在生产环境中，有时会需要将两台主机的特定目录实现实时同步。比如，将NFS共享目录的数据文件自动同步到备份服务器特定目录中。

**实现实时同步的两种方法**

- inotify + rsync 方式实现数据同步
- sersync ：前金山公司周洋（花椒直播）在 inotify 软件基础上进行开发的，功能更加强大，不过有一定时间未更新

**实现方式：**

- 使用inotify服务监控文件发生的变化
- 利用rsync服务推送到备份服务器

**实现inotify软件：**

- inotify-tools
- sersync
- lrsyncd

### 4.2 inotify+rsync+shell 脚本实现实时数据同步

**实验环境**

```bash
10.0.0.2 数据服务器
10.0.0.3 备份服务器
```

**具体执行步骤**

```bash
10.0.0.2
#安装inotify-tools
[root@centos8 ~]# yum -y install inotify-tools
[root@centos8 ~]# yum -y install rsync
[root@centos8 ~]# cat inotify_rsync.sh 
#!/bin/bash
SRC='/data/www/'  
DEST='rsyncuser@10.0.0.3::backup'
rpm -q rsync &> /dev/null || yum -y install rsync

inotifywait  -mrq  --exclude=".*\.swp" --timefmt "%Y-%m-%d %H:%M:%S" --format '%T %w %f' -e create,delete,moved_to,close_write,attrib ${SRC} |while read DATE TIME DIR FILE;do
        FILEPATH=${DIR}${FILE}
	       rsync -az --delete  --password-file=/etc/rsync.pas $SRC $DEST && echo
	       "At ${TIME} on ${DATE}, file $FILEPATH was backuped up via rsync" >> /var/log/changelist.log
       done
[root@centos8 ~]# echo "123321" > /etc/rsync.pas
[root@centos8 ~]# mkdir /data/www
[root@centos8 ~]# chmod 600 /etc/rsync.pas

#测试能否正常访问
[root@centos8 ~]# rsync -avz --delete --password-file=/etc/rsync.pas /data/www/ rsyncuser@10.0.0.3::backup
sending incremental file list

sent 113 bytes  received 12 bytes  250.00 bytes/sec
total size is 0  speedup is 0.00
[root@centos8 ~]# bash inotify_rsync.sh

10.0.0.3
#安装服务包
[root@centos8 ~]# dnf install rsync-daemon
[root@centos8 ~]# vim /etc/rsyncd.conf
uid = root
gid = root
#use chroot = no
max connections = 0
ignore errors
exclude = lost+found/
log file = /var/log/rsyncd.log
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsyncd.lock
reverse lookup = no
#hosts allow = 10.0.0.0/24
[backup]  
path = /data/backup/  
comment = backup dir		#描述
read only = no     
auth users = rsyncuser  #指定rsyncuser用户才能访问
secrets file = /etc/rsync.pas	#指定用户名密码存放位置
[root@centos8 ~]# mkdir /data/backup
[root@centos8 ~]# echo "rsyncuser:123321" > /etc/rsync.pas
[root@centos8 ~]# chmod 600 /etc/rsync.pas
```