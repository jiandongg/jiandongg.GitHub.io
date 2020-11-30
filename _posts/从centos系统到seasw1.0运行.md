1. 分配cpu隔离

   目的：cpu隔离之后，程序可以独占cpu性能，同时提高cpu cache的命中率。

   分配方法：修改配置文件grub，见相关笔记。

   注意事项：对于多numa架构的机器，cpu隔离时尽量不要跨numa，使程序运行集中在一个numa中

2. 分配大页内存

   目的：大页内存使得程序可以独占内存，提高程序性能。

   分配方法：修改配置文件grub，见相关笔记。

   注意事项：大页内存的分配以2M或者1G为单位进行分配，分配时的参数可能有所不同，注意辨别。

   2.5 安装依赖包：`yum`，最好是联网，或者是使用光盘源。见`使用光盘源作为yum源`笔记。

   `yum -y upgrade`

   `yum -y update`

   需要用rpm独立安装的libpcap、libpcap-devel库：

   `rpm -ivh xxx.rpm`

   svn可能也需要这么安装，如果有互联网连接，可以直接`yum -y install svn`

   万不得已，可以使用rpm安装，但有依赖不全的风险。

   2.6 拷贝源代码，善用svn，lsblk，mount，umount，eject命令

3. 编译DPDK

   进入dpdk文件夹

   `cd xxxx/dpdk`

   删除冗余文件

   `rm -rf x86_64........../`

   `make clean -j64`

   设定环境变量

   `vi dpdk.rc`

   `source dpdk.rc`（内部路径注意更新）

   ~~配置并编译DPDK~~
   ~~`make config T=x86_64-native-linuxapp-gcc && make`~~

   安装DPDK

   `make install T=x86_64-native-linuxapp-gcc -j64`

   -j64调用并行核心，提高make速度
   此时会在dpdk文件夹下产生新的x86_64-native-linuxapp-gcc文件夹，编译安装过程完成。随后开始绑定网卡（将网卡绑定给编译好的DPDK程序组建）
   (
      `modprobe uio`

      `insmod ${RTE_SDK}/x86_64-native-linuxapp-gcc/kmod/igb_uio.ko`
   )

3. 绑定网卡

   目的：绑定网卡归DPDK所用

   分配方法：执行相关脚本命令，见相关笔记

   注意事项：执行脚本时注意目录切换，不要重复执行，重启则绑定的网卡失效

4. 编译lib

   进入lib文件夹，lib文件夹中存放着程序的配置信息，比如/lib/pof_common/rte_pof.h就有关于是否开启抓包的宏定义

   `cd xxx/lib`

   如果需要进行O3优化，进行性能测试，则注意修改Makefile文件，使O3优化生效
   `vi Makefile` 
   `vi pof_common/Makefile` 
   `vi pof_pipeline/Makefile` 
   `vi pof_table/Makefile` 

   清空，编译

   `make clean`

   `make`

5. 编译app

   进入app文件夹

   `cd xxx/app`

   如果需要进行O3优化，进行性能测试，则注意修改Makefile文件，使O3优化生效
   `vi Makefile` 

   清空，编译

   `make clean`

   `make`

6. 检查seasw.cli

   `/seasw1.0/seasw.cli`

   seasw.cli为配置文件，用来指导seasw的工作

7. 根据seasw.cli配置文件所述，检查.asm文件，并生成ex.bin

   `seasw1.0/app/asm/`

   .asm为控制器流表配置文件，ex.bin为其生成的格式文件，使用asmpof生成

   `./seasw1.0/app/asm/asmpof xxxxx.asm ex.bin`

   

8. 执行seasw主程序

`./build/app/seasw_fe -c 0x55 -- -s seasw.cli`

-c 0x55 为程序执行调用的cpu的十六进制掩码，1代表调用，0代表不调用

--为分隔符

-s seasw.cli 为程序引用的配置文件

执行后的打印，也表明了程序的工作流程，比如LINK个数，加载组建等等信息。

此后程序会进行正式的运行，并依照流表进行数据包转发工作

9. 使用telnet连接交换机

`telnet 127.0.0.1 8086`

10. 程序日志文件

`vi ./seasw1.0/app/seasw.log`

11. 关于SINK_PCAP的宏配置文件

12. 表项删除的两个主题：控制器下发命令或FE核主动删；表项超时FE核删除的两种方式：硬超时（表项安装到超时）、软超时（数据包表项命中到超时）

13. 打开或关闭pcap抓包的宏定义：seasw1.0m/lib/pof_common/rte_pof.h

14. 代码中的pad[x]用于凑齐n64字节，提高内存管理效率

15. 打开关闭自定义acl算法：/lib/pof_table/rte_pof_table_mm.c 开头的#define MYACL

16. debug方法：函数屏蔽（return 0或者//或者#if 0 #endif）、telnet连接、延迟打印（每9999轮打印一次）、逐个包重放


