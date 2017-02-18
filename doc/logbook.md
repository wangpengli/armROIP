BBB Fix Log
==

LED指示说明：
> USR0 is configured at boot to blink in a heartbeat pattern
USR1  is configured at boot to light during microSD card accesses
USR2 is configured at boot to light during CPU activity
USR3 is configured at boot to light during eMMC access

## 2017.02.18

### # 001

插入SD卡按住boot button上电，四个LED全不亮，串口输出CCCC

### # 002

查[资料](https://zhidao.baidu.com/question/265753308801132765.html)发现板子默认禁止了eMMC的烧写功能，需要把板子自带系统中修改/boot/uEnv.txt文件
```
##enable BBB:eMMC Flasher:
#cmdline=init=/opt/scripts/tools/eMMC/init-eMMC-flasher-v3.sh
```
修改为
```
##enable BBB:eMMC Flasher:
cmdline=init=/opt/scripts/tools/eMMC/init-eMMC-flasher-v3.sh
```
查看uEnv.txt后发现其中相应内容是
```
##enable Generic eMMC Flasher:
#cmdline=init=/opt/scripts/tools/eMMC/init-eMMC-flasher-v3.sh
```
修改为
```
##enable Generic eMMC Flasher:
cmdline=init=/opt/scripts/tools/eMMC/init-eMMC-flasher-v3.sh
```
后插入SD卡，按住boot button启动，四个LED灯同时常亮，串口输出四个C后无输出。

### # 003

怀疑修改uEnv.txt文件时改错行，就从SD卡启动系统，挂载eMMC查看文件，发现没有改错。
没做修改退出后，断电重新插SD卡，按住boot button启动，四个LED灯全不亮，串口持续输出CCCC。

### # 004

修改过uEnv.txt不插SD卡启动的话，串口会打印以下提示：
```
-----------------------------
Starting eMMC Flasher from microSD media
Version: [1.20160222: deal with v4.4.x+ back to old eeprom location...]
-----------------------------
Checking for Valid bbb-eeprom header
Valid bbb-eeprom header found [335]
-----------------------------
copying: [/dev/mmcblk0] -> [/dev/mmcblk1]
lsblk:
NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
mmcblk0boot0 179:8    0   16M  1 disk 
mmcblk0boot1 179:16   0   16M  1 disk 
mmcblk0      179:0    0  3.6G  0 disk 
`-mmcblk0p1  179:1    0  3.6G  0 part /
-----------------------------
df -h | grep rootfs:
-----------------------------
Error: [/dev/mmcblk1] does not exist
writing to [/dev/mmcblk1] failed...
[   15.276496] Kernel panic - not syncing: Attempted to kill init! exitcode=0x00
000200
[   15.276496] 
[   15.285690] CPU: 0 PID: 1 Comm: init-eMMC-flash Not tainted 4.4.6-ti-r15 #1
[   15.292680] Hardware name: Generic AM33XX (Flattened Device Tree)
[   15.298837] [<c0015b61>] (unwind_backtrace) from [<c00123f5>] (show_stack+0x1
1/0x14)
[   15.306629] [<c00123f5>] (show_stack) from [<c03c8643>] (dump_stack+0x73/0x80
)
[   15.313893] [<c03c8643>] (dump_stack) from [<c00f22bf>] (panic+0x87/0x1b4)
[   15.320805] [<c00f22bf>] (panic) from [<c0034285>] (do_exit+0x7d5/0x7f4)
[   15.327537] [<c0034285>] (do_exit) from [<c00342fd>] (do_group_exit+0x2d/0x8c
)
[   15.334791] [<c00342fd>] (do_group_exit) from [<c003436f>] (SyS_exit_group+0x
13/0x14)
[   15.342665] [<c003436f>] (SyS_exit_group) from [<c000ec41>] (ret_fast_syscall
+0x1/0x52)
[   15.350713] ---[ end Kernel panic - not syncing: Attempted to kill init! exit
code=0x00000200
[   15.350713] 
[   29.820309] random: nonblocking pool is initialized
```
按下power button关闭电源后，串口打印：
```
U-Boot SPL 2016.03-00001-gd12d09f (Mar 17 2016 - 16:16:15)
Trying to boot from MMC
bad magic
```

## 总结

现在情况是SD卡插入，不按boot button，自动从SD卡启动。
不管修改不修改配置文件来打开或禁用eMMC烧写，都无法向eMMC烧写系统。

在[另一篇文章](http://blog.thestargazer.net/201407/%E5%BD%93beaglebone-black%E5%8F%98%E7%A0%96%E5%A4%B4%E6%97%B6-%E5%86%99%E7%BB%99%E9%82%A3%E4%BA%9B%E5%92%8C%E6%88%91%E4%B8%80%E6%A0%B7%E5%80%92%E9%9C%89%E7%9A%84%E6%9C%8B%E5%8F%8B/)里提到了一个从usb向eMMC烧写镜像的方法：

>思路就是把放在usb里面的img直接拷贝到整个emmc上就ok了，
当然如果想要用Angstrom的话可以恢复下缺失的文件，比如MLO
拷贝方法也是很单纯
fatload usb 0 ${loadaddr} ubuntuXXXX.img (bytes) (pos)   这里的byte pos都是HEX的
然后
mmc write ${loadaddr} (blackstart) (count)
这样就可以把镜像拷贝过去了，可惜镜像比较大，得多分几次拷贝，没办法啊
分次拷贝的时候 可以先看img文件大小 在excel里面计算好，写成script
然后teraterm里面复制到命令行里面执行
大概20分钟就拷贝完后就可以直接从emmc启动ubuntu了
