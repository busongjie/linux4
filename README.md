## boot下的内核配置文件  
vim /boot/grub/grub.conf

## 看设备和目录的对应关系  
centos7：losetup  
centos6：losetup -a   

## 分区的挂载根目录下都有个lost+found  
 
## mount命令  
-r 只读（一般挂载重要分区）  
-n 不会更新此文件（起到隐藏挂载的效果）  
-a 自动挂载所有支持自动挂载的设备(与/etc/fstab相关）  
-L 'LABEL' 以卷标指定挂载设备  
-U 'UUID'以UUID指定挂载的设备  
-B，--bind 绑定目录到另一个目录上  
mount /boot /mnt/boot         --bind  
cat /proc/mounts  
查看内核追踪到的已挂载的所有设备  
-o（挂载文件系统的选项），多个选项使用逗号分隔  
async 异步模式（更快）  
sync 同步模式（更安全）内存更改时，同时写磁盘  
atime/noatime 包含目录和文件  
diratime/nodiratime 目录的访问时间戳    
auto/noauto 是否支持自动挂载,是否支持-a选项  
exec/noexec是否支持将文件系统上运行应用程序  
dev/nodev是否支持在此文件系统上使用设备文件  
suid/nosuid是否支持suid和sgid权限  
ro只读  
rw读写  
user/nouser是否允许普通用户挂载此设备，/etc/fstab使用  
acl启用此文件系统上的acl功能  
loop使用loop设备  
defaults：相当于rw, suid, dev, exec, auto, nouser, async  
*（有用）remount重新挂载（不取消挂载的情况下直接添加功能）  

## 去除系统的acl  
(1)tune2fs -o ^acl /dev/sdb1  
(2)mount -o remount,noacl /dev/sdb1  
 
## 自动加acl  
(1)tune2fs -o acl /dev..  
(2)mount -o rmount,acl /mnt/sdb1  

## 生成100M大小的文件  
dd if=/dev/zero of=/data/exit4file bs=1M count=100  

## 格式化文件系统  
mkfs.exit4 /data/ext4file  

## blkid看文件系统的信息  

## centos6挂载文件  
mount -o loop ...  

## 卸载命令  
查看挂载情况  
*findmnt MOUNT_POINT|device  
看此目录有谁在用  
(1)*lsof /mnt/sdb1  
(2)fuser -v /mnt/sdb1  
终止所有正在访问指定的文件系统的进程  
(慎用)fuser -km /mnt/sdb1  
强行结束后加-f   
卸载  
umount  

## （重点）写到/etc/fstab实现真正的挂载（一切皆文件思想）  
1.vim /etc/fstab打开  

2.设备标识 挂载点 文件系统 挂载选项 最后一项文件系统检查命令  

3.（1）改完后，没挂载的可以用mount -a  
 （2）已经挂载修改信息重新启动用  
mount -o rmount /mnt...     
  
## 如何知道系统报错信息  
cat /var/log/boot.log  

## 生成/dev/sdb1的UUID的命令  
blkid /dev/sdb1  

## swap（交换分区）是模拟内存使用的  

## 如何将4个G的swap扩展到8个G并且永久生效  
1.分区fdisk /dev/sdd  
2.创建文件系统mkswap /dev/sdd1  
3.vim /etc/fstab改一下  
free -h看大小  
swapon -s查看当前哪些swap设备正在生效  
swapon -a针对swap启用命令  

## 调优先级  
1.vim /etc/fstab 将defaults换成pri=大于-1的数  
2.swapoff /dev/sdd1禁用swap  
3.swapon -a 启用  
临时启动起来并指 定优先级  
swapon-p 5 /data/swapfile   

## 如何以文件方式提供swap  
1.先建个2G的文件  
dd if=/dev/zero of=/swapfile bs=2G count=1  
2.格式化  
mkswap /swapfile  
3.进vim /etc/fstab修改  
4.swapon -a启用  
5.swapon -s看一下正在启用的swap  
6.（改不改不影响使用）  
chmod 600 /swapfile
7.swapon -s在确认一下  

## 将swapfile文件迁移到别的磁盘中  
1.先禁用swapoff /swapfile  
2.移动 mv /swapfile /data/  
3.进vim /etc/fstab中改路径  
4.swapon -a启用  
5.看一下 swapon -s  

## 删除swap  
1.先禁用swapoff /data/swapfile /dev/sdd1  
2.进vim中删除加swap的行  
3.删除文件  
rm -rf /data/swapfile  
删除分区  
进入fdisk /dev/sdd （p、d、w）  
结束  

## 如何把home迁移到一个独立的分区  
1.先分区fdisk /dev/sda(p、n、+10G、p、w)  
2.同步 partprobe  
3.创建文件系统  
mkfs.xfs /dev/sda7  
blkid看下文件系统是否生成   
4.建一个临时目录  
mkdir /mnt/home  
5.挂载上去  
mount /dev/sda7 /mnt/home  
6.将home中数据拷贝到/mnt/home  
cp -av /home/. /mnt/home  
看一下   
ls /mnt/home/  
ll /mnt/home/  
du -sh /home  
du -sh /mnt/home  
7.永久挂载vim /etc/fstab  
mount -a 启用  
看一下  
df-h  
ls /home  

## 如何删除/home迁移  
1.取消挂载  
umount /home  
2.清除旧数据  
rm -rf /home/*  

## eject命令卸载或弹出磁盘  
## eject -t 弹入磁盘  

## 把光盘制作成ISO镜像  
cp /dev/sr1 /data/centos6.iso  
使用时挂载  
mount /data/centos6.iso /mnt/iso  
把etc的目录下打包成 /data/etc.iso  
mkisofs -r -o /data/etc.iso /etc/  
看文件格式  
file /data/etc.iso  
挂载  
mkdir /mnt/etc  
mout /data/etc.iso /mnt/etc  
注意是只读文件  

## dmesg命令可以看硬件信息  

## 看usb设备的情况  
1.lsusb可以查到usb的信息  
2.tail /var/log/messages  
通过日志可以看到硬盘的信息  

## 常见工具  
文件系统空间占用等信息的查看工具：  
df[OPTION]...[FILE]...  
-H 以1000为单位  
-T 文件系统类型  
-h：human-readable  
-i（节点）：inodes instead of blocks  
-P：以Posix兼容的格式输出  

## 查看某目录总体空间占用状态  
du -h：human-readable  
du -s：summary --max-depth   


## 工具dd  
dd命令：convert and copy a file  
用法：  
dd if=/PATH/FROM/SCR of=/PATH/TO/DEST  
bs=#:block size,复制单元大小  
count=#：复制多少个bs  
of=file 写到所命名的文件而不是到标准输出  
if=file 从所命名文件读取而不是从标准输入  
bs=size 指定块大小（既是ibs也是obs）  
ibs=size 一次读size个byte  
obs=size 一次写size个byte  
cbs=size 一次转化size个byte  
skip=blocks 从开头忽略blocks个ibs大小的块  
seek=blocks  
从开头忽略blocks个obs大小的块  
count=n 只拷贝n个记录  
conv=conversion[,conversion...] 用指定的参数转换文件  
转换参数：  
ascii转换EBCDIC为ASCII  
ebcdic转换ASCII为EBCDIC  
lcase把大写字符转换为小写字符  
ucase把小写字符转化为大写字符  
nocreat不创建输出文件  
noerror 出错时不停止  
notrunc 不截短输出文件  
sync把每个输出块填充到ibs个字节，不足部分用空（NUL）字符补齐  
Fdatasync 写完成前，物理写入输出文件  
## 备份MBR  
dd if=/dev/sda of=/tmp/mbr.bak bs=512 count=1  
## 破坏MBR中的bootloader  
dd if=/dev/zero of=/dev/sda bs=64 count=1 seek=446  
## 有一个大于2k的二进制文件fileA。现在想从第64个字节位置开始读取，需要读取的大小是128Byts。又有fileB，想把上面读取到的128Bytes写到第32个字节开始的位置，替换128Bytes，显示如下：  
dd if=fileA of=fileB bs=1 count=128 skip=63 seek=31 conv=notrunc  

## 销毁磁盘数据  
dd if=/dev/urandom of=/dev/sda1  
利用随机的数据填充硬盘，在某些必要的场合可以用来销毁数据，执行此操作以后/dev/sda1将无法挂载，创建和拷贝操作无法执行  

## 测试硬盘写速度  
dd if=/dev/zero of=root/1Gb.file bs=1024 count=1000000
## 测试硬盘读速度  
dd if=/root/1Gb.file bs=64k|dd of=/dev/null  
 
 ## 修复磁盘  
 dd if=/dev/sda of=/dev/sda   
 
## 小插曲  
cat >f1  
abcdefghi  
cat > f2  
123456  
dd if=f2 of=f1 bs=1 count=2 count=2 skip=3 seek=4  
cat f1   
abcd45  
dd if=f2 of=f1 bs=1 count=2 count=2 skip=3 seek=4 conv=notrunc  
cat f1  
abcd45ghi  
 

## RAID  
RAID级别  
RAID-0：条带卷，strip  
RAID-1：镜像卷，mirror  
RAID-2  
...  
RAID-5  
RAID-6  
RAID-10   
RAID-01  

## RAID-0：（用的不是很多了）  
成员大小一致  
读写性能提升  
可用空间：N*min（s1,s2,...）  
无容错能力  
最少磁盘数：2，2+  

## RAID-1:  
读性能提升、写性能略有下降  
可用空间：1*min（s1,s2,...）  
有冗余能力  
最少磁盘数：2，2N  
注意:RAID-1不能起到恢复删库  
只能起到保护硬件的作用  

## RAID-4:  
多块数据盘异或运算值存于专用校验盘  

## RAID-5：（用的最多）  
读、写性能提升  
可用空间：（N-1）*min（s1,s2,...）  
有容错能力：允许最多1块磁盘损坏  
最少磁盘数：3，3+  

## 空闲硬盘技术 spare disk技术   
RAID-6：  
读、写性能提升  
可用空间：（N-2）*min（s1,s2,...）  
有容错能力：允许最多2块磁盘损坏  
最少磁盘数：4，4+  
 
## RAID-10：（先做1后做0 ）  
读、写性能提升  
可用空间：N*min(s1,s2,...)/2  
有容错能力：每组镜像最多只能坏一块  
最少磁盘数：4，4+  
RAID-01：（先做0后做1）  
01和10相比，10容错性好  

## 常用RAID级别有：  
RAID-1,RAID-5,RAID-10  

## 软RAID  
注意每个磁盘取出的空间要一致  
每块硬盘不需要单独的创建文件系统  
1.看一下取出的磁盘空间保证是没用的  
2.删除文件系统  
blkid看一下  
rm -f /data/  
清理磁盘  
dd if=/dev/zreo of=/dev/sdb bs=1 count=512  
注意同步问题：  
partx -d --nr 1 /dev/sdb  
看一下（会发现删的并不干净）  
hexdump -c /dev/sdb -v -n  
102400   
3.分区  
fdisk /dev/sdb  
(n、1、+2G、t（改类型）、fd、p、w)  
 克隆一下(工作中不建议用)  
 dd if=/dev/sdb of=/dev/sdc  
 bs=1 count=512  
 lsblk看一下  
 4.生效partx -a /dev/sdc  
 5.创建RAID-0  
 mdadm -c -a yes /dev/md0 -l 0 -n 2 /dev/sd{b,c}1  
 ll /dev/md0看一下是否生成  
 cat /proc/mdstat  
 看一下RAID状态  
  mdadm -D可以更详细的看RAID的状态  
  6.创建文件系统  
  mkfs.ext4 /dev/md0
  blkid 看文件系统
  7.挂载
  mkdir /mnt/raid
  vim /etc/fstab添加
  8.启动生效
  ##  创建RAID-5
   1.分区  
   fdisk /dev/sdb  
   fdisk /dev/sdc  
   fdisk /dev/sdd  
   2.生效  
   partx -a /dev/sdb  
   partx -a /dev/sdc  
   partx -a /dev/sdd  
    3.mdadm -C -a yes /dev/md1 -1 5 -n 3 -x 1 -c 1M /dev/sd{b2,c2,d1,e1}   
    4.创建文件系统  
    mkfs.ext4 /dev/md1  
    5.进vim /etc/fstab修改挂载  
    6.创建文件夹  
    mkdir /mnt/raid2  
    7.启用  
    mount -a  
    df -h看一下  
    8.将备用硬盘存放到配置文件中  
    madam -Ds > /etc/mdadm.conf
    
    模拟sdd1失败（没有真正失败）
    mdadm /dev/md1 -f /dev/sdd1
    mdadm -D /dev/md1
    如何给它恢复回去
    mdadm /dev/md1 -r /dev/sdd1
    mdadm /dev/md1 -a /dev/sdd1
    sdd1成为备用的了
    
    禁用RAID用
    mdadm -S /dev/md1
    启用RAID用
    mdadm -A /dev/md1
    
    删除 RAID  
    1.禁用RAID  
    2.删除 RAID超级块  
    mdadm --zero-superblock /dev/sdb  
    3.删除分区  
    dd if=/dev/zero of=/dev/sdc bs=1 count=512  
    4.同步  
    partx -d --nr 1-2 /dev/sdc  
    partx -d --nr 1-3  
    /dev/sdb  
    5.进入vim /etc/fstab删除完事  
  
  
 ##  逻辑卷（LVM）  
1.划分分区  
fdisk /dev/sda(n、+10G、p、t、6、L、8e、q、w)  
lsblk看一下分区情况   
partx-a /dev/sda同步  

2.pvs看有没有  
pvdisplay看的更详细  
pvcreate /dev/sd{a6,c}转化成物理卷  

3.vgs看有没有卷组  
vgdiplay看卷组详情  
vgcreate vg0 /dev/sd{a6,c} 创建卷组  

4.lvs看有没有逻辑卷  
lvdisplay看逻辑卷详情  
lvcreate -n lv_mysql -L 15G vg0  
到此逻辑卷的创建完成了  

## 使用逻辑卷前的准备  
创建文件系统  
mkfs.ext4 /dev/vg0/lv_mysql  
blkid 看文件系统   
mkdir /mnt/mysql  
mount /dev/vg  
mount /dev/vg0/lv_mysql /mnt/mysql  
df-h  

## 看效果  
1.dd if=/dev/zero of=mnt/mysql/db1 bs=1M count=1024  
dd if=/mnt/mysql/db1 of=/dev/null  
 
## 逻辑卷的扩展：（前提先看一下卷组的空间）  
1.看逻辑卷空间  
vgdisplay  
2.把剩下空间都用光  
lvextend -l +100%FREE /dev/vgo/lv_mysql  
pvdisplay  
df -h（第一次看到的是空间里文件系统的大小）  
df -hT  
3.在线创建文件系统  
resize2fs /dev/vg0/lv_mysql（这个命令有局限性只适应于ext系统）  
xfs_growfs /mnt/mysql(xfs系统用这个命令)   
df -hT  
## 如果卷组没有空间了，怎么办  
1.利用热插拔技术，加一块硬盘  
2.创建物理卷  
pvcreat /dev/sdb  
pvs看一下  
3.扩展卷组  
vgextend vg0 /dev/sdb  
vgdisplay看一下卷组的情况  

## 缩减逻辑卷(记住先备份,慎用)  
1.取消挂载   
umount /mnt/mysql/  
2.检查系统完整性  
e2fsck -f /dev/vg0/lv_mysql   
3.缩减文件系统  
resize2fs /dev/vg0/lv_mysql 20G  
4.缩减逻辑卷  
lvreduce -L 20G /dev/vg0/lv_mysql  
5.挂载  
mount /dev/vg0/lv_mysql /mnt/mysql/  

## 逻辑卷的隔机迁移  
1.用pvdisplay命令看一下是否还有其他硬盘有空间  
2.将/dev/sda6移出  
pvmove /dev/sda6  
3.将/dev/sda6从vg0中移走  
pvdisplay  
vgreduce vg0  /dev/sda6  
4.删除分区  
fdisk /dev/sda(p、d、6、w)  
5.同步  
partx -d -nr 6 /dev/sda  
 pvdisplay看一下物理卷  
6. 压缩空间  
（1）取消挂载  
 umount /mnt/mysql  
（2）检查文件系统完整性  
 fsck -f /dev/vg0/lv_mysql  
（3）缩减文件系统  
 resize2fs /dev/vg0/lv_mysql 10G   
（4）缩减逻辑卷   
 lvreduce -L 10G /dev/vg0/lv_mysql  
（5）挂载  
 mount /dev/vd0/lv_mysql /mnt/mysql/  
 df -h 看一下分组情况  
 7.从卷组中删除/dev/sdb  
vgreduce vg0 /dev/sdb  

（两个逻辑卷组名重了怎么办   
vgrename vg0 newvg0改名  
1.取消挂载  
umount /mnt/mysql/  
2.设为禁用状态  
vgchange -an newvg0  
3.导出  
vgexport newvg0  
lvdisplay看一下  
vgdisplay）  

8.centos6拔硬盘时，看一下 pvdisplay   
centos7加硬盘  
scandisk启用硬盘  
9.导入vgimport /mntnewvg0  
vgchange -ay newvg0激活  
10.挂载  
mkdir /mnt/mysql  
mount /dev/newvg0/lv_mysql /mnt/mysql  

## *扩展一步到位(那个系统都可以用)  
lventend -r -L 5G /dev/vg0/lv_data   

## 逻辑卷快照（不能代替备份，一般用于测试）  
目的：快速对数据做备份  
原理：只有在文件在改动的情况下，才备份原文件，否则不备份，注意：只备份最早的原文件  

## centos6创建快照  
1.lvcreate -n lv_mysql_snap -s -p r -L 1G /dev/newvg0/lv_mysaql   
2.blkid  
3.mkdir /mnt/snap  
mount /dev/newvg0/lv_mysql_snap /mnt/snap  
  
## centos6还原  
1.取消挂载，取消快照  
umount /mnt/mysql/  
umount /mnt/snap/  
2.lvconvert --merge(合并） /dev/newvgo/lv_mysql_snap  
3.数据还原  
mount /dev/newvg0/lv_mysql /mnt/mysql/  
centos7还原和centos6一样  

## centos7创建快照  
1.lvcreat -n lv_data_snap -s -p r -L 1G /dev/vg0/lv_data 
2.blkid  
3.mkdir /mnt/snap  
2.mount -o nouuid /dev/vg0/lv_data_snap /mnt/snap/  
 
 ## 删除快照  
 先取消挂载  
 umount /mnt/mysql  
 umount /mnt/snap  
 删除快照  
 lvremove /dev/newvg0/lv_mysql  
