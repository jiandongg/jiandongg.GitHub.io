0.在boot界面，使用u盘安装系统时，可以得知系统是legacy（老式bios）或者是uefi（新式）启动。！！！

以下**针对CentOS7的uefi启动方式**，挂载1GB的大页内存：

1.创建大页内存挂接点 
mkdir /mnt/huge_1GB 
mount -t hugetlbfs nodev /mnt/huge_1GB

2.在/etc/fstab文件中加入如下命令，使其重启后有效 
nodev /mnt/huge_1GB hugetlbfs pagesize=1GB 0 0

3.修改配置文件中启动菜单的内核参数

vi /boot/efi/EFI/centos/grub.cfg

4.查找关键字”menuentry”启动项，在代码段的特定位置添加“default_hugepagesz=1G hugepagesz=1G hugepages=16”，每一处的都要加上。注意做好原文件备份

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

5.在配置文件目录下执行grub2命令，使其配置在系统的合适位置生效，生成新的启动文件

cd /boot/efi/EFI/centos/

grub2-mkconfig -o /boot/grub2/grub.cfg

显示done代表配置完成

6.重启即可，查看大页内存配置情况

`grep -i huge /proc/meminfo`

`cat /proc/cmdline`

备注：对于legacy的普通bios启动方法，可能修改的文件是/etc/default/grub，注意目录的切换（grub2）和更改方式（linux_cmdline="xxxx"）的改变

备注：如果配置cpu隔离，则加上以下描述：

 isolcpus=1-4 nohz_full=1-4 rcu_nocbs=1-4

备注：使用lscpu查看cpu情况，其中

Threads per core：每个核有几个线程

CPU Socket：有几个物理CPU

core per socker：每个物理CPU有几个核心

NUMA nodes : Non Uniform Memory Access Architecture，使众多服务器像单一系统那样运转，几个NUMA

CPUs：有几个逻辑的处理器，=物理CPU个数*每个物理CPU核心个数*每个核心的线程个数

其中，我们能够配置CPU隔离的参数是核心单位