**0.在boot界面，使用u盘安装系统时，可以得知系统是legacy（老式bios）或者是uefi（新式）启动。！！！**

A、

以下**针对CentOS7的uefi启动方式**，挂载1GB*16的大页内存：

注意：不要把过多内存都配置为大页内存，防止影响linux自身性能

1.创建大页内存挂接点 
`mkdir /mnt/huge_1GB`
`mount -t hugetlbfs nodev /mnt/huge_1GB`

2.在/etc/fstab文件末尾加入如下命令，使其重启后有效 
`nodev /mnt/huge_1GB hugetlbfs pagesize=1GB 0 0`

3.修改配置文件中启动菜单的内核参数，注意做好原文件备份

`cd /boot/efi/EFI/centos/`

`cp grub.cfg majd_grub.cfg`

`vi grub.cfg`

4.查找关键字”menuentry”启动项（在vim的命令模式输入`:/menuentry`，回车，按`n`查找下一项）。在menuentry代码段的特定位置添加“default_hugepagesz=1G hugepagesz=1G hugepages=16”，对每一个menuentry的代码段中都要加上。

```
### BEGIN /etc/grub.d/10_linux ###
menuentry 'CentOS Linux (3.10.0-1127.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-1127.el7.x86_64-advanced-d648124d-f16b-4f63-a074-aef1f7bc06fc' {
	load_video
	set gfxpayload=keep
	insmod gzio
	insmod part_gpt
	insmod xfs
	set root='hd0,gpt2'
	if [ x$feature_platform_search_hint = xy ]; then
	  search --no-floppy --fs-uuid --set=root --hint-bios=hd0,gpt2 --hint-efi=hd0,gpt2 --hint-baremetal=ahci0,gpt2  60ad466f-e10a-44c2-b1b1-61218721cd36
	else
	  search --no-floppy --fs-uuid --set=root 60ad466f-e10a-44c2-b1b1-61218721cd36
	fi
	linuxefi /vmlinuz-3.10.0-1127.el7.x86_64 root=UUID=d648124d-f16b-4f63-a074-aef1f7bc06fc ro default_hugepagesz=1G hugepagesz=1G hugepages=16 crashkernel=auto spectre_v2=retpoline console=ttyS0,115200 
	initrdefi /initramfs-3.10.0-1127.el7.x86_64.img
}
menuentry 'CentOS Linux (0-rescue-83cb142ebb1948a684b72eb8bf140996) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-0-rescue-83cb142ebb1948a684b72eb8bf140996-advanced-d648124d-f16b-4f63-a074-aef1f7bc06fc' {
	load_video
	insmod gzio
	insmod part_gpt
	insmod xfs
	set root='hd0,gpt2'
	if [ x$feature_platform_search_hint = xy ]; then
	  search --no-floppy --fs-uuid --set=root --hint-bios=hd0,gpt2 --hint-efi=hd0,gpt2 --hint-baremetal=ahci0,gpt2  60ad466f-e10a-44c2-b1b1-61218721cd36
	else
	  search --no-floppy --fs-uuid --set=root 60ad466f-e10a-44c2-b1b1-61218721cd36
	fi
	linuxefi /vmlinuz-0-rescue-83cb142ebb1948a684b72eb8bf140996 root=UUID=d648124d-f16b-4f63-a074-aef1f7bc06fc ro default_hugepagesz=1G hugepagesz=1G hugepages=16 crashkernel=auto spectre_v2=retpoline console=ttyS0,115200 
	initrdefi /initramfs-0-rescue-83cb142ebb1948a684b72eb8bf140996.img
}
if [ "x$default" = 'CentOS Linux (3.10.0-1127.el7.x86_64) 7 (Core)' ]; then default='Advanced options for CentOS Linux>CentOS Linux (3.10.0-1127.el7.x86_64) 7 (Core)'; fi;
### END /etc/grub.d/10_linux ###
```

5.**在配置文件目录下**执行grub2命令，使其配置在系统的合适位置生效，生成新的启动文件

`cd /boot/efi/EFI/centos/`（即刚修改的grub.cfg文件所在的位置）

`grub2-mkconfig -o /boot/grub2/grub.cfg`（在该路径下会生成新的启动文件，可通过`ll`查看修改时间验证）

显示done代表配置完成

6.!!!重启生效!!!，查看大页内存配置情况

`reboot`!!!一定要记得，重启，才能生效

`grep -i huge /proc/meminfo`

`cat /proc/cmdline`

备注：**对于legacy的普通bios启动方法**，可能修改的文件是/etc/default/grub，注意当下目录的切换（执行`grub2-mkconfig -o /boot/grub2/grub.cfg`命令时的路径应改为/etc/default/grub）和更改方式（linux_cmdline="xxxx"）的改变。

注意：带内存插槽的机器在配置`grub2`命令时可能会出现以下警告：

```
[   69.579272] sr 0:0:0:0: [sr0] CDROM not ready.  Make sure there is a disc in the drive.
[   69.589013] sr 0:0:0:1: [sr1] CDROM not ready.  Make sure there is a disc in the drive.
[   69.598646] sr 0:0:0:2: [sr2] CDROM not ready.  Make sure there is a disc in the drive.
[   69.608276] sr 0:0:0:3: [sr3] CDROM not ready.  Make sure there is a disc in the drive.
```

属于正常现象，不影响配置命令本身的执行效果。

注意：在虚拟机上执行大页内存划分时，可能会因大页内存划分过多导致`HugePages_Total`比设定值少的情况。此时加大该虚拟机的分配内存即可

B、

**配置cpu隔离**1-4号逻辑核，则在后面加上以下描述：

`isolcpus=1-4 nohz_full=1-4 rcu_nocbs=1-4`

注意：尽量不要隔离0号逻辑cpu、不要将过多cpu都隔离，以免影响linux自身性能。

注意：使用lscpu查看cpu情况，其中

Threads per core：每个核有几个线程

CPU Socket：有几个物理CPU

core per socker：每个物理CPU有几个核心

NUMA nodes : Non Uniform Memory Access Architecture，使众多服务器像单一系统那样运转，几个NUMA

CPUs：有几个逻辑的处理器，=物理CPU个数*每个物理CPU核心个数*每个核心的线程个数

其中，我们能够配置CPU隔离的参数是核心单位。

注意：单numa机器`lscpu`

```
[majd@localhost ~]$ lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                4
On-line CPU(s) list:   0-3
Thread(s) per core:    1
Core(s) per socket:    1
Socket(s):             4
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 158
Model name:            Intel(R) Core(TM) i7-9750H CPU @ 2.60GHz
Stepping:              10
CPU MHz:               2591.661
BogoMIPS:              5183.32
Hypervisor vendor:     VMware
Virtualization type:   full
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              12288K
NUMA node0 CPU(s):     0-3
```

对于单numa机器，隔离`1,2`与隔离`2,3`并无二致

双numa机器`lscpu`

```
[root@seasw ~]# lscpu
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                32
On-line CPU(s) list:   0-31
Thread(s) per core:    2
Core(s) per socket:    8
Socket(s):             2
NUMA node(s):          2
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 85
Model name:            Intel(R) Xeon(R) Silver 4208 CPU @ 2.10GHz
Stepping:              7
CPU MHz:               2100.000
BogoMIPS:              4200.00
Virtualization:        VT-x
L1d cache:             32K
L1i cache:             32K
L2 cache:              1024K
L3 cache:              11264K
NUMA node0 CPU(s):     0,2,4,6,8,10,12,14,16,18,20,22,24,26,28,30
NUMA node1 CPU(s):     1,3,5,7,9,11,13,15,17,19,21,23,25,27,29,31
Flags:...
```

对于多numa机器，应注意将cpu隔离选择的逻辑核编号选择在同一个物理cpu中，即选择`2,4,6,8,10,12`的性能要优于`2,3,4,5,6,7`。

因为跨numa，即跨物理cpu需要更远距离的信息传输。

