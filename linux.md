#Linux
##Linux文件系统结构
- Linux文件系统为一个倒转的单根树状结构
- 文件系统的根为`"/"`
- 文件系统严格区分大小写
- 路径使用`"/"`分割，（windows中使用`\`）
![linux_tree](http://img.blog.csdn.net/20130830202418671)
![tree](http://paching.myweb.hinet.net/picture/dirtree.png)


##SHELL
	用户<=>Shell | Kernel	
![Kernel](http://h.hiphotos.baidu.com/baike/pic/item/8601a18b87d6277fe955984828381f30e924fc7f.jpg)

- Shell 分为CLI与GUI两种
- CLI: Command Line Interface
- GUI: Grophical User Interface

###操作系统的Shell
- GUI: GNOME
- CLI: BASH

###BASH
- 提示符：#，$
- 命令一般由三个部分组成：*命令、选项、参数*
- 使用Tab键来简化命令输入
	* 自动补全命令
	* 自动补全文件名
	* 无法自动补全参数

###创建、删除文件
- 通过`touch`命令可以创建一个空文件或更新文件时间
- 通过`rm`命令可以删除文件或目录
- 常用参数：
	* -i 交互式
	* -r 递归的删除包括目录中的所有内容
	* -f 强制删除，没有警告提示（使用时需十分谨慎）

##磁盘基本概念
- cylinder(柱面)
- sector (扇区)
- head (磁头)
![cylinder](http://img5.imgtn.bdimg.com/it/u=1466118564,3574593327&fm=21&gp=0.jpg)
![single-platter View](http://www.dnzg.cn/uploads/allimg/140511/211RI5A-6.png)

##磁盘在Linux中的表示
- Linux所有设备都被抽象为一个文件，保存在`/dev`目录下。
- 设备名称一般为hd[a-z] (a-z为区号)，
- IDE设备的名称为hd[a-z],SATA, SCSI, SAS, USB等设备的名称为hd[a-z]


##分区概念
	将一个磁盘逻辑的分为几个区，每个区当作独立磁盘，以方便使用管理。
	
- 不同分区用：设备名称+分区号 方式表示，如：sda1,sda2.
- **主流的分区机制分为MBR和GPT两种**

###MBR
- MBR(Master Boot Record)是传统的分区机制，（应用于绝大多数使用BIOS的PC设备）。
- MBR支持32bit和64bit系统
- MBR只支持分区数量有限
- MBR只支持不超过2T的硬盘，超过2T的硬盘将只能使用2T空间（有第三方解决方法）

###MBR结构

![MBR_Struct](http://plcdn.qiniudn.com/wp-content/uploads/2012/01/MBR-Struct.png)

###MBR分区
- 主分区
	最多只能创建4个主分区
- 扩展分区
	一个扩展分区会占用一个主分区的位置	
- 逻辑分区
	Linux最多支持63个IDE分区和15个SCSI分区。
![MBR_room](http://img.my.csdn.net/uploads/201204/05/1333611609_2604.PNG)

###GPT
	GPT (GUID Partition Table)是一个较新的分区机制，解决了MBR的很多缺点。
- 支持超过2T的磁盘
- 向后兼容
- 必须在支持UER的硬盘上才能使用
- 必须使用64bit系统
- Mac, Linux系统都能支持GPT分区格式
- Windows7 64bit、windowsServer 2008 64bit支持GPT

###FDISK分区工具
	fdisk是来自IBM的老牌分区工具，支持绝大多数操作系统，几乎所有的Linux的发行版本都装有fdisk,包括在Linux的`rescue`模式下的依然能够使用。
- fdisk是一个基于MBR的分区工具，所以如果需要使用GPT，则无法使用fdisk进行分区。
- fdisk命令只有具有超级用户权限才能够运行
- 使用fdisk -l可以列出所有安装的磁盘及其分区信息
- 使用`fdisk /dev/sda` 可以对目标磁盘进行分区操作
- 分区之后需要使用partprabe命令让内核更新分区信息，否则需要重启才能识别新的分区
- /proc/partitions文件也可用来查看分区信息

###文件系统
	操作系统通过文件系统管理文件及数据，磁盘或分区需要创建文件系统之后才能够为操作系统使用，创建文件系统的过程又称为格式化。
- 没有文件系统的设备称之为裸（raw）设备
- 常见的文件系统有（fat32、ext2、ext3、ext4、xfs、HFS）等<font color=red size=2>（看不清楚，待确认）</font>
- 文件系统之间的区别：日志、支持的分区大小、支持的单个文件大小、性能等
- windows 下的主流文件系统是：NTFS Linux下的主流文件系统是：Ext3、Ext4

###Linux支持的文件系统
- `ext2`, `ext3`, `ext4`, `fat(msdos)`, `vfat`, `nfs`, `iso9660`, `proc`, `gfs`, `jfs`

###MKE2FS
	命令`make2fs`用来创建文件系统`make2fs -t ext4 /dev/sda3`
####常用命令
- `-b blocksize` 指定文件系统块大小
- `-c` 建立文件系统时检查坏损块
- `-L tabel` 指定卷标
- `-j` 建立文件系统日志

###JOURNAL日志
	带日志的文件系统(ext3、ext4)拥有较强的稳定性，在出现错误时可以进行恢复。
- 使用带日志的文件系统，文件系统会使用一个叫做`两阶段提交`的方式进行磁盘操作，当进行磁盘操作时，文件系统进行以下操作：
- 文件系统将准备执行的事务的具体内容写入日志
- 文件系统进行操作
- 操作成功后，将事务的具体内容从日志中删除
- 这样的好处是，当事务执行的时候如果出现意外（如断电或磁盘故障），可以通过查询日志进行恢复操作，缺点是会丧失一定的性能（额外的日志读写操作）。

###FSCK
	命令fsck用来检查并修复损坏的文件系统`fsck /dev/sda2`
- 使用`-y`参数不提示而直接进行修复
- 默认fsck会自动判断文件系统类型，如果文件系统损坏较为严重，请使用`-t`参数指定文件系统类型。
- 对于识别为文件的损坏数据（文件系统无记录），fsck会将该文件放入`lost+found`目录
- 系统启动时会对磁盘进行fsck操作

###挂载操作
	磁盘或分区创建好文件系统后，需要挂载到一个目录才能够使用。windows或Mac会进行自动挂载，一旦创建好文件系统后会自动挂载到系统上，windows上称之为C盘、D盘等。Linux需要手工进行挂载操作或配置系统进行自动挂载。
![Linux_挂载](http://hub9.lxway.net/upload/2/a3/2a325502d4a90cba16ae5c10ff21203a.png)
###MOUNT
	在Linux中，我们通过`mount`命令将格式化好的磁盘或分区挂载到一个目录上。
	eg：mount /dev/sda3(要挂载的分区)  /mnt（挂载点）

####常用参数
- 不带参数的`mount`命令会显示所有已挂载的文件系统
- `-t` 指定文件系统的类型
- `-o` 指定挂载选项
- `ro, rw`以只读或读写塔式挂载，默认是rw
- `sync`代表不使用缓存，而是对所有操作直接写入磁盘
- `async`代表使用缓存，默认是`async`
- `noatime`代表每次访问文件时不更新文件的访问时间
- `atime`代表每次访问文件时更新文件的访问时间
- `remount`重新挂载文件系统

###UMOUNT
	命令umount用来卸载已挂载的文件系统，相当于windows中的弹出
	eg: umount 文件系统/挂载点
	    umount /dev/sda3 == umount /mnt


- 如果出现device is busy报错，则表示该文件系统正在被使用，无法卸载。
- 可以使用以下命令查看使用文件系统的进程：`fuser -m /mnt`
- 也可以使用命令`lsof`查看正在使用的文件:`lsof /mnt`

###自动挂载

配置文件`/etc/fstab`用来定义需要自动挂载的文件系统，fstab中每一行代表一个挂载配置，格式如下：

|/dev/sda3|/mnt  |ext4    |defaults|o o     |
|---------|:----:|:------:|:------:|:------:|
|需要挂载的设备|挂载点|文件系统|挂载选项|dump、fsck相关选项|

- 要挂载的设备也可以使用LABEL进行识别，使用LABEL=LINUXCAST取代/dev/sda3
- `mount -a`命令挂载所有`fstab`中定义的自动挂载项

###获取帮助
####没必要记住所有东西
	Linux提供了极为详细的帮助工具及文档，一定要养成查看帮助查文档的习惯，可以大大减少需要记忆的东西并且提高效率。
	
###MAN
- `man`命令是Linux中最为常用的帮助命令，将要获取帮助命令作为参数运行`man`命令就可以获取相应的文档帮助
- `man`文档分为很多类型：

|部分  |类型|
|:----:|:-------------------:|
|1|用户命令|
|2|系统调用|
|3|库函数|
|4|设备文件|
|5|文件格式|
|6|游戏|
|7|帮助手册|
|8|系统管理|
|9|内核|

- `man -k`关键字 可以用来查询包含该关键字的文档

###INFO

- `info`与`man`类似，但是提供的信息更为详细深入，以类似网页的形式显示
- `man`与`info`都可以都过`/+关键字`方式进行搜索

###DOC
	很多程序、命令都带有详细的文档，以txt、HTML、PDF等方式保存在/usr/share/doc目录中，这些文档是相应程序最为详尽的文档
	
##用户、组
	当我们使用Linux时，需要以一个用户的身份登入，一个进程也需要以一个用户的身份运行，用户限制使用者或进程可以使用、不可以使用哪些资源。
	组用来方便组织管理用户
	
- 每个用户拥有一个`UserID`,操作系统实际使用的是用户ID，而非用户名
- 每个用户属于一个主组，属于一个或多个附属组
- 每个组拥有一个GroupID
- 每个进程以一个用户身份运行，并受该用户可访问的资源限制
- 每个可登陆用户拥有一个指定的`shell`

###用户
- 用户ID为32位，从0开始，但是为了和老式系统兼容，用户ID限制在60000以下。
- 用户分为以下三种：
	- root用户 （ID为0的用户为root用户）
	- 系统用户 （1~499）
	- 普通用户 （500以上）
- 系统中的文件都有一个所属用户及所属组
- 使用`id`命令可以显示当前用户的信息
- 使用`passwd`命令可以个性当前用户密码

###相关文件
- `/etc/passwd` 保存用户信息
- `etc/shadow` 保存用户密码（加密的）
- `/etc/group` 保存组信息

###查看登陆的用户
- 命令`whoami`显示当前用户
- 命令`who`显示有哪些用户已经登陆系统
- 命令`w`显示有哪些用户已经登陆并且在干什么

###修改用户信息
- 命令`usermod`用来修改用户信息
	- `usermod`参数 `username`
- 命令usermod支持以下参数
	- `-i` 新用户名
	- `-u` 新`userid`
	- `-d` 用户家目录位置
	- `-g` 用户所属主组
	- `-G` 用户所属附属组
	- `-L` 锁定用户例其不能登陆
	- `-U` 解除锁定
	
###组
	几乎所有操作系统都有组的概念，通过组，我们可以更加方便的归类、管理用户。一般来讲，我们使用部门、职能或地理区域的分类方式来创建使用组。
- 每个都有一个组ID
- 组信息保存在`/etc/group`中
- 每个用户都有一个主组，同时还可以拥有最多31个附属组


###创建、修改、删除组
- 命令`groupadd`用以创建组：
	- `groupadd linux_xi` 
- 命令`groupmod`用以修改组信息：
	- `groupmod -n newname oldname` 修改组名
	- `groupmod -g newGid oldGid` 修改组ID
	
- 命令`groupdel`用以删除组：
	- `groupdel linux_c`


***


###默认权限
- 每一个终端都拥有一个`umask`属性，来确定新建文件、文件夹的默认权限
- `umask`使用数字权限方式表示，如：022
- 目录的默认权限是：777 -umask
- 文件的默认权限是：666 -umask
- 一般，普通用户的默认`umask`是002，root用户的默认`umask`是022
- 也就是说，对于普通用户来说：
- 新建文件的权限是：666-002 = 664
- 新建目录的权限是：777-002 = 775
- 命令`umask`用以查看设置`umask`值


###特殊权限
|权限|对文件的影响|对目录的影响|
|:-----:|:---------:|:----------:|
|suid|以文件的所属用户身份执行，而非执行文件的用户|无|
|sgid|以文件所属组身份执行|在该目录中创建的任意新文件的所属组与该目录的所属组相同|
|sticky|无|对目录所有写入权限的用户仅可以删除其拥有的文件，无法删除其他用户所拥有的文件|


###设置特殊权限
- 设置suid:
	- `chmod u+s linux`
- 设置sgid:
	- `chmod g+s linux`
- 设置sticky:
	- `chmod o+t linux`

- 与普通权限一样，特殊权限也可以使用数字方式表示
	- `suid = 4`
	- `sgid = 2`
	- `sticky = 1`
- 所以，我们可以通过以下命令设置：
	- `chmod 4775 linux`

***
##IP编址
- ip编址是一个双层编址方案，一个ip地址标识一个主机（或一个网卡接口）
- 现在应用最为广泛的是`ipv4`编址，已经开始逐渐向`ipv6`编址切换
- `ipv4`地址为32位长，`ipv6`地址为128位长
- 一个`ipv4`地址分为两个部分：网络部分和主机部分
- 网络部分用来标识所属区域、主机部分用来标识该区域中的哪个主机

||	|	|
|:-------:|:---------:|:---------:|
|32bit|网络部分|主机部分|

***

###域名
- IP地址往往难以记忆，所以我们一般使用域名进行管理
	- eg: **www.fengniao.com**
域名分为三个部分，用`.`分割：
- 类型	标识此域名的类型（`com/net/org`）
- 域名	域名称
- 主机名		该域中的某台主机名称

|主机名| |域名| |类型|
|:---:|:---:|:----:|:---:|:---:|
|www|.|Linux|.|com|

- 域名大小写不敏感

###DNS
- 每个域名代表一个IP，而DNS服务器就是用来在IP与域名之间进行转换的。
- `www.fengniao.com` <=> `61.1.1.1`
- DNS服务由DNS服务器提供
![DNS](http://s3.51cto.com/wyfs02/M02/22/C7/wKiom1Mm7quBr-LjAAIGC-_bV2A797.jpg)

###基本网络参数
- 要配置一个局域网通信的计算机：
	- IP地址
	- 子网掩码
- 要配置一个跨网段通信的计算机：
	- IP地址
	- 子网掩码
	- 网关
- 要配置一个可上网的计算机：
	- IP地址
	- 子网掩码
	- DNS

###以太网连接
- 在Linux中，以太网接口被命令为：eth0,eth1等，0，1代表网卡编号
- 通过`lspci`命令可以查看网上硬件信息（如果是usb网卡，则可能需要使用`lsusb`命令）
- 命令`ifconfig`命令用来查看接口信息
	- `ifconfig -a`	 查看所有接口
	- `ifconfig eth0` 查看特定接口
	
- 命令`ifup`,`ifdown`用来启用、禁用一个接口
	`ifup eth0`
	`ifdown eth0`

###网络测试命令
- 测试网络连通性：
	- `ping 192.168.1.1`
	- `ping www.csdn.com`
	
- 测试DNS解析：
	- `host www.linux.com`
	- `dig www.linux.com`
- 显示路由表：
	- `ip route`

- 追踪到达目标地址的网络路径：
	`traceroute www.linux.com`
- 使用`mtr`进行网络质量测试(结合了`traceroute`和`ping`)
	- `mtr www.linux.com`

###修改主机名
- 实时修改主机名：
	- `hostname train.linux.com`
- 永久性修改主机名：
	- `/etc/sysconfig/network`
	**HOSTNAME=train.linux.com**

###故障排查
- 网络故障排查遵循<font color=red size=2>**从底层到高层，从自身到外部**</font>的流程进行

- 先查看网络配置信息是否正确：
	- IP地址
	- 子网掩码
	- 网关
	- DNS
- 查看到达网关是否连通
	- `ping`网关IP地址
- 查看DNS解析是否正常：
	- `host www.linux.com`
	- `host www.126.com`
	- `host www.douban.com`

###日期时间
- 命令`date`用以查看、设置当前系统时间：
- 命令`hwclock(clock)`用以显示硬件时钟时间
- 命令cal用以查看日历
- 命令uptime用以查看系统运行时间

###输出、查看命令
- 命令 

***

###关机、重启
- 命令`shutdown`用以关闭，重启计算机
	- **`shutdown`[关机、重启] 时间**
	- `-h`关闭计算机
	- `-r`重新启动

- 如：
	- 立即关机：**`shutdown -h now`**
	- 10分钟关机：**`shutdown -h +10`**
	
	- 23:30分关机：**`shutdown -h 23:30`**
	- 立即重启：**`shutdown -r now`**
	
- 命令`poweroff`用以立即关闭计算机
- 命令`reboot`用以立即重启计算机

### 归档、压缩
- 命令`zip`用以压缩文件
	- **`zip linux.zip myfile`**
- 命令`unzip`用以解压缩zip文件
	- **`unzip linux.zip`**
- 命令`gzip`用以压缩文件
	- **`gzip linux`**
- 命令`tar`用以归档文件
	- **`tar -cvf out.tar linux`**
	- **`tar -xvf linux.tar`**
	- **`tar -cvzf backup.tar.gz /etc`**
		- `-z`参数将归档后的归档文件进行`gzip`压缩减少大小
- 命令`locate`用以快速查找文件、文件夹：
	- `locate keyword`
- 此命令需要预先建立数据库，数据库默认每天更新一次，可用`updated`命令手工建立、更新数据库
- 命令`find`用以高级查找文件、文件夹：
	- `find 查找位置 查找参数`

- 如：
	- **`find . -name *linux*`**
	- **`find / -name *.conf`**
	- **`find / -perm 777`**
	- **`find / -type d`**
	- **`find . -name "a*" -exec ls -i {}\ `**

###FIND查找条件
- `find`支持很多种的查找条件，常用的如下：
	- `-name`
	- `-perm`
	- `-user`
	- `-group`
	- `-ctime`
	- `-type`
	- `-size`
	
##不要重复发明轮子

###管道和重定向
	在linux系统中，大多数命令都很简单，很少出现复杂功能的命令，每个命令往往只实现和一个或几个很简单的功能，我们可以通过将不同功能的命令组合在一起使用，以达到完成某个复杂功能的目的。

- Linux中，几乎所有命令的返回数据都是纯文本的（因为命令都是运行在CLI下），而纯文件形式的数据又是绝大多数命令的输入格式，这就让多命令协作成为可能。
- Liunx的命令行为我们提供了管道和重定向机制，多命令协作就是通过管道和重定向完成的。
***

### 







	