---
title: Linux常用操作
date: 2024-02-09 16:46:50
tags: 
    - Linux
    - tools
categories:
    - Linux
keywords: 
    - Linux
---


# ArchLinux

## optimus-manager

* 在终端使用命令切换显卡
* 先运行 prime-offload

```shell
# 切换为英伟达显卡
optimus-manager --switch nvidia
# 切换为intel核显
optimus-manager --switch intel
```

```shell
# 无效
sudo prime-select nvidia # 切换nvidia显卡
sudo prime-select intel  # 切换intel显卡
sudo prime-select query  # 查看当前使用的显卡
```

> journalctl -p 3 -xb 查看系统日志



* glxinfo命令可以查看当前正在使用的显卡。

```shell
# 切换前默认Intel
[rainerosion@rains-arch ~]$ glxinfo | grep "OpenGL renderer"
OpenGL renderer string: Mesa DRI Intel(R) HD Graphics 4600 (HSW GT2)
# 切换NVIDIA显卡后
[rainerosion@rains-arch ~]$ glxinfo | grep "OpenGL renderer"
OpenGL renderer string: GeForce GTX 950M/PCIe/SSE2
  
```

* 查看驱动版本信息

```shell
ls /usr/src | grep nvidia
```

* 查看显卡

```shell
lspci -k | grep -A 2 -E "(VGA|3D)"
```

## 切换nvidia驱动后重启无法进入图形界面
> ```markdown
> 按ctrl+alt+f2进入命令行，输入用户名密码进入（建议），输入命令nvidia-smi查看显卡是否运行正常
> 
> 运行nvidia-xconfig生成配置
> 输入lspci | grep -E "VGA|3D"查看你的显卡PCI地址，
> 将类似图中每行最前面的00.02.0转换成PCI:0:2:0，填在下面代码块中BusID那里，我使用的是NVIDIA，按照图中应该为PCI:1:0:0。将下面代码块添加到/etc/X11/xorg.conf文件中（同理，如果你是使用nvidia-xconfig后无法进入图形界面，你可以（删除/etc/X11/xorg.conf来重新进入图形界面）谨慎操作！！！）：
> Section "Module"
>     Load "modesetting"
> EndSection
> 
> Section "Device"
>     Identifier "nvidia"
>     Driver "nvidia"
>     BusID "PCI:1:0:0"
>     Option "AllowEmptyInitialConfiguration"
> EndSection
> 
> 编辑/usr/share/sddm/scripts/Xsetup文件，将下列内容添加到文件中
> xrandr --setprovideroutputsource modesetting NVIDIA-0
> xrandr --auto
> 
>
> ```

## 切换回intel

> sudo vim /etc/optimus-manager/optimus-manager.conf
>
> 修改 startup_mode=integrated  后 注销


## 登录选择位置

> /usr/share/xsessions



## nmcli 连接wifi

> nmcli dev 查看网络设备
>
> nmcli r wifi on 打开wifi
>
> nmcli r wifi off 关闭wifi
>
> nmcli dev wifi 扫描wifi 

> 1. 创建wifi连接
>
> $ nmcli device wifi connect {wifi名} password {密码}
>
> 2. 删除wifi连接
>
> $ nmcli con del {wifi名}
>
> 3. 启用wifi连接
>
> $ nmcli connection up {wifi名}
>
> $ nmcli device con {网卡名}
>
> 4. 查看已有连接
>
> $ nmcli con

* 若报错`Error: Connection activation failed: (7) Secrets were required, but not provided.` 删除连接记录

```shell
$ nmcli c 
NAME                UUID                                  TYPE      DEVICE
eno1                3e755f37-3cff-314d-bb1b-4efebb6ce566  ethernet  eno1
OpenWrt             14ad438b-9b2b-43c6-a3d1-66dd3bb987dc  wifi      --
$ nmcli c delete OpenWrt
Connection 'OpenWrt' (14ad438b-9b2b-43c6-a3d1-66dd3bb987dc) successfully deleted.
$ nmcli device wifi connect "OpenWrt" password 00000000
Device 'wlp58s0' successfully activated with '180a3ce4-ba1f-405a-94f1-684e538a7be9'.
```

但是隐藏了SSID的则需要额外操作，如下：

```
nmcli c add type wifi con-name "自定义连接名称" ifname "无线网卡名称" ssid "WiFi的名字"
nmcli con modify "自定义连接名称" wifi-sec.key-mgmt wpa-psk
nmcli con modify "自定义连接名称" wifi-sec.psk "WiFi的密码"
nmcli con up "自定义连接名称"
```

查看网卡信息：

```shell
# 查看网卡信息：
nmcli dev show wlp7s0

# 创建网卡
nmcli connection add type bridge ifname br0 stp no

# 查看桥接绑定信息
brctl show
```



## 创建证书连接

> ```shell
> nmcli connection add \
>  type wifi con-name "IMUNET-NG" ifname wlp7s0 ssid "IMUNET-NG" -- \
>  wifi-sec.key-mgmt wpa-eap 802-1x.eap ttls \
>  802-1x.phase2-auth mschapv2 802-1x.identity "32309140" \
>  802-1x.password "131539"
>  
>  
> 启动时(第一次)：
> nmcli --ask connection up IMUNET-NG
> ```

## nmcli  创建网桥

注意: 仅有线网卡可以

1. 添加新的网桥： `nmcli connection add type bridge con-name virbr0 ifname virbr0 autoconnect yes`
2. 创建子网卡： `nmcli con add type bridge-slave ifname ${网卡名} master ${网桥名}`
3. 打开 br0： `nmcli con up br0`
4. 



## 查看端口

netstat -anp | grep 端口号

lsof  -i: 端口号

ps -ef 查看进程

w , tty

pkill 命令：杀死终端中所有命令 - pkill -9 -t pts/0



## patch补丁

>  Reversed (or previously applied) patch detected! Assume -R? [n]

* `-t`：该参数遇到这种情况直接将打过补丁的文件恢复原样，即未打补丁之前的状态
* `-f`：该参数遇到这种情况则继续打补丁，当然一般情况下会报错，毕竟对比不一致了
* `-N`：忽略该文件

基本命令语法：  
　　patch [-R] {-p(n)} [--dry-run] < patch_file_name  
　　p：为path的缩写。  
　　n：指将patch文件中的path第n条’/’及其左边部分取消。  
　　-R：卸载patch包。  
　　--dry-run：尝试patch软件，并不真正修改软件。

　　例如：

　　$ patch -p1 < /home/sense/patch-2.6.19.1      安装补丁  
　　$ patch -p1 -R < /home/sense/patch-2.6.19.1 卸载补丁

## DPI设置

```shell
# ~/.Xresources
# 默认96
Xft.dpi: 192
xrdb -merge /home/papillon/.Xresources
```

应用程序缩放：

> --force-device-scale-factor=2 



## 双屏设置

> xrandr --output eDP-1-1 --primary --mode 3840x2160 --scale 1x1 --output DP-0 --mode 1920x1080 --scale 2x2 --left-of eDP-1-1 



## 向日葵启动

>sudo systemctl start runsunloginclient.service 
>
>sudo systemctl enable runsunloginclient.service

## chrome 代理

1. `sudo google-chrome --proxy-server="127.0.0.1:8080" --no-sandbox`
1. 使用代理插件

## 调整透明度

> * xprop 点击窗口获取窗口信息 WM_CLASS(STRING) 后的内容
> * vim ~/.config/picom/picom.conf 
> * 在透明度中新加一行更改指定透明度



## 调节音量

alsamixer

> pgup/pgdowm 增减音量
>
> 1~9 调节音量到 10%-90%

## 调节屏幕亮度

```shell
# 手动切换
切换root用户
cd /sys/class/backlight/intel_backlight
echo 0~512 > brightness

sudo pacman -S acpilight
sudo gpasswd video -a 用户名 # 将当前用户添加到video实现免root控制亮度

# 获取当前亮度
xbacklight -get
# 设置亮度
xbacklight -set 70
# 增加亮度
xbacklight -inc 10
# 减少亮度
xbacklight -dec 10

```

## 切换内核

```shell
# 修改 grub 配置不是必须的，下面那个才是必须的。
$ sudo nano /etc/default/grub
# 这三行是将子菜单展开，这样不用点击 advanced 进去了
GRUB_DISABLE_SUBMENU=y
GRUB_DEFAULT=saved
GRUB_SAVEDEFAULT=true
# 等待时间我之前设置为 0s 的，0 就不显示了，这里改大一点 3s
GRUB_TIMEOUT=3

# 改完保存退出 -----------------------------------------


# 更新下 grub 的配置文件，让上面的设置生效，这一步是一定要做的。
$ sudo grub-mkconfig -o /boot/grub/grub.cfg
正在生成 grub 配置文件 ...
找到 Linux 镜像：/boot/vmlinuz-linux-lts
找到 initrd 镜像：/boot/amd-ucode.img /boot/initramfs-linux-lts.img
Found fallback initrd image(s) in /boot:  amd-ucode.img initramfs-linux-lts-fallback.img
找到 Linux 镜像：/boot/vmlinuz-linux
找到 initrd 镜像：/boot/amd-ucode.img /boot/initramfs-linux.img
Found fallback initrd image(s) in /boot:  amd-ucode.img initramfs-linux-fallback.img
警告： os-prober will not be executed to detect other bootable partitions.
Systems on them will not be added to the GRUB boot configuration.
Check GRUB_DISABLE_OS_PROBER documentation entry.
Adding boot menu entry for UEFI Firmware Settings ...
完成

```

## 救援模式
linux系统奔溃后的紧急救援模式
当 Linux 系统奔溃时，可以进入紧急救援模式（emergency mode）来检查和修复系统问题。以下是进入紧急救援模式的步骤：

在启动 GRUB 时，选择要启动的 Linux 内核。

按下 e 键进入编辑模式，在 kernel 行的末尾添加 single 或 emergency。

按下 Ctrl + X 启动系统。

系统会自动进入救援模式，此时会提示输入 root 用户密码，输入密码后即可进入救援模式。

在救援模式下，可以进行以下操作：

检查文件系统：使用 fsck 命令检查文件系统，并修复错误。

挂载文件系统：使用 mount 命令挂载文件系统，使得可以访问文件系统中的文件。

修改密码：使用 passwd 命令修改用户密码。

网络配置：使用 ifconfig 命令配置网络。

安装软件：使用 yum 或 apt-get 命令安装软件。

退出救援模式：使用 exit 命令退出救援模式，重新启动系统。


## 系统

- **关机**

```
shutdown -h 0 #<==O秒后关机
shutdown -h now #<==现在关机
shutdown -h 10 #<==10分钟后关机
shutdown -h 23:20 #<==23：20分关机
shutdown -c #<==取消shutdown关机命令
init 0 #<==立马关机（切换运行级别为0，推荐使用）
halt #<==立马关机
poweroff #<==立马关机

```

- **重启**

```
shutdown -r now #<==现在重启
shutdown -r 23:20 & #<==23：20分重启，加&符号代表把该命令转到后台处理
reboot #<==立马重启（推荐使用）
init 6 #<==立马重启（切换运行级别为6，推荐使用）
```

- **注销**

```
logout #<==立马注销
exit #<==立马注销
在SecureCRT软件中，按快捷键：Ctrl + d #<==推荐使用
```

## 切换java版本

```shell
# 查看archlinux-java使用说明
$ archlinux-java --help

# 查看jdk状态
$ archlinux-java status

Available Java environments:
  java-11-openjdk
  java-8-openjdk (default)

# 获取默认jdk
$ archlinux-java get

java-8-openjdk

# 设置默认jdk
$ sudo archlinux-java set java-11-openjdk

# 查看切换后的jdk版本
$ java -version

openjdk version "11.0.13" 2021-10-19
OpenJDK Runtime Environment (build 11.0.13+8)
OpenJDK 64-Bit Server VM (build 11.0.13+8, mixed mode)

```

## crontab脚本

> - -e : 执行文字编辑器来设定时程表，内定的文字编辑器是 VI，如果你想用别的文字编辑器，则请先设定 VISUAL 环境变数来指定使用那个文字编辑器(比如说 setenv VISUAL joe)
> - -r : 删除目前的时程表
> - -l : 列出目前的时程表
>
> ```
> min hour day mon week program
> ```
>
> | 每分钟定时执行一次       | * * * * * |
> | ------------------------ | --------- |
> | 每小时定时执行一次       | 0 * * * * |
> | 每天定时执行一次         | 0 0 * * * |
> | 每周定时执行一次         | 0 0 * * 0 |
> | 每月定时执行一次         | 0 0 1 * * |
> | 每月最后一天定时执行一次 | 0 0 L * * |
> | 每年定时执行一次         | 0 0 1 1 * |

*  要确保有对目标文件的权限
*  命令要使用绝对路径

## Top

```txt
c： 显示完整的命令
d： 更改刷新频率
f： 增加或减少要显示的列(选中的会变成大写并加*号)
F： 选择排序的列
h： 显示帮助画面
H： 显示线程
i： 忽略闲置和僵死进程
k： 通过给予一个PID和一个signal来终止一个进程。（默认signal为15。在安全模式中此命令被屏蔽）
l:  显示平均负载以及启动时间（即显示影藏第一行）
m： 显示内存信息
M： 根据内存资源使用大小进行排序
N： 按PID由高到低排列
o： 改变列显示的顺序
O： 选择排序的列，与F完全相同
P： 根据CPU资源使用大小进行排序
q： 退出top命令
r： 修改进程的nice值(优先级)。优先级默认为10，正值使优先级降低，反之则提高的优先级
s： 设置刷新频率（默认单位为秒，如有小数则换算成ms）。默认值是5s，输入0值则系统将不断刷新
S： 累计模式（把已完成或退出的子进程占用的CPU时间累计到父进程的MITE+ ）
T： 根据进程使用CPU的累积时间排序
t： 显示进程和CPU状态信息（即显示影藏CPU行）
u： 指定用户进程
W： 将当前设置写入~/.toprc文件，下次启动自动调用toprc文件的设置
<： 向前翻页
>： 向后翻页
?： 显示帮助画面
1(数字1)： 显示每个CPU的详细情况


```

## scp

```
#上传本地文件夹到Linux
scp -r -p 22 {本地文件夹} Linux用户名@ip地址:Linux绝对路径
#上传文件
scp -p 22 {本地文件} Linux用户名@ip地址:Linux绝对路径

# 上传
去掉指定端口号参数-p, 并将上述路径舒徐调换即可

#下载
scp -p 22 linux用户名@ipdizhi:文件路径 {本地路径}
```

## ln 软链接

ln -s 【目标目录】 【软链接地址】

目标目录】指软连接指向的目标目录下，【软链接地址】指“快捷键”文件名称，该文件是被指令创建的。

rm -rf 【软链接地址】

ln -snf 【新目标目录】 【软链接地址】

## ROFI Script

### 格式

> rofi -show [mine] -modi "mine:~/scripts/rofi.sh"

返回选中的文本

> ls | rofi -dmenu 
>
> history | rofi -dmenu.    h=$(history | rofi -dmenu)



## 屏蔽磁盘

1　启动后的禁用 无需重启

　（sdx是你的磁盘 udev的更新可能会导致磁盘重新出现 在向系统添加/删除磁盘也可能会改变）

> ```javascript
>  echo 1>/sys/block/sdx/device/delete 
> ```

2　基本上你创建一个文件`/etc/udev/rules.d/99-hide-disks.rules` ，你在其中放置该行

> ```ini
>  KERNEL=="sda1", ENV{UDISKS_PRESENTATION_HIDE}="1" 
> ```

　 其中*sda1*是您要隐藏的分区的名称。 在某些系统中它也可以

> ```ini
>  KERNEL=="sda1", ENV{UDISKS_IGNORE}="1" 
> ```

   然后你重新启动。



## QQ闪退

> 文件问题： sudo chattr +i ~/.config/QQ/crash_files/
>
> 代理问题： 不要走代理



# pacman

## 强制安装

>pacman -S --overwrite '*' 要覆盖的文件模式**

## 在 Arch Linux 上安装 Debtap

>要安装 Debtap，启动终端并使用 AUR 助手安装它：
>
>```
>yay -S debtap
>sudo debtap -u
>```
>
>使用 cd 命令进入 DEB 文件的目录并使用 Debtap 开始转换包。
>
>```
>cd ~/Downloads
>debtap yourfile.deb
>```
>
>Debtap 将创建一个“**你的文件.zst**” 文件，您可以使用包管理器轻松安装，在本例中为 pacman。
>
>```
>sudo pacman -U yourfile.zst
>```

## 使用dpkg

>1. 安装dpkg
>2. 使用dpkg安装deb包
>
>```
>yay -S dpkg
>dpkg -i .deb包名
>```

## 安装

pacmam -U package.pkg.tar.gz 从安装本地package包



## 多线程编译

```SHELL
sudo vim /etc/makepkg.conf 
```

找到 MAKEFLAGS 参数,修改 -j 后面的参数为想要用多少个核编译,保持并退出

```tex
MAKEFLAGS="-j8"
```



## 代理问题

```shell
# 设置 http 代理
export http=http://127.0.0.1:7890
export https=http://127.0.0.1:7890

# 或, 设置 socket 代理(clash)
export http_proxy=socks5://127.0.0.1:7891
export https_proxy=socks5://127.0.0.1:7891
```

# Shell

## 注意

> 带多个参数的命令 要加 command 修饰，例如
>
> `command gcc -g main.c -o main`




# 内核编译
## qemu安装
1. i386:  qemu-system-i386
2. x86_64: qemu-system-x86_64
3. arm : qemu-system-arm
4. arm64: qemu-system-aarch64
## 交叉编译链安装
1. arm-linux-gnueabihf-
2. aarch64-linux-gnu-
## 编译linux内核
> make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
> make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
> make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j 8

## 制作跟文件系统 busybox
1. 下载源码，并设置对应目标的架构，编译。
2. qemu-img create xxx.img 4g 创建4g磁盘，使用mkfs.ext4 xxx.img 格式化
```shell
# 创建 1g (这个根据实际需要进行配置) 的 raw 格式的空镜像磁盘 
qemu-img create rootfs.img 1g 
# 将创建的磁盘格式化成 ext4 类型的文件系统 
mkfs.ext4 rootfs.img 
# 创建一个文件用来挂载该镜像（用来将编译好的内容装入磁盘，载体） 
mkdir rootfs 
# 将 rootfs.img 挂载到 rootfs 文件夹 
sudo mount -o loop rootfs.img rootfs 
# 将在 busybox 中编译的内容移动到刚新建的文件夹 rootfs 中(等价于拷贝到了 rootfs.img 磁盘中) 
cd rootfs 
sudo cp -r ../busybox/_install/* . 
# 补充一些必要的文件夹 
sudo mkdir proc sys dev etc etc/init.d

cd etc/init.d/ sudo touch rcS sudo vi rcS

# 文件内容如下
#!/bin/sh
mount -t proc none /proc	# 挂载文件系统到 /proc 目录, 它提供了一个接口来访问内核和进程信息
mount -t sysfs none /sys	# 挂载sysfs文件系统到 /sys 目录, 提供了一种方式来访问和调整内核的运行时信息
/sbin/mdev -s				# 使用mdev（是BusyBox的一部分）来初始化/dev目录，它负责创建设备节点
							# -s 告诉 mdev 启动是扫描 /sys 目录并创建设备节点 
							# /sys中包含了当前系统中硬件设备的详细信息


sudo chmod +x rcS

cd ../..
sudo umount rootfs
```

## qemu启动系统
`qemu-system-aarch64 -M virt -cpu cortex-a57 -m 256M -nographic -kernel ../linux-6.7/arch/arm64/boot/Image -drive file=rootfs.img,format=raw,id=hd0,if=none -device virtio-blk-device,drive=hd0 -append "root=/dev/vda rw console=ttyAMA0"
`
## 虚拟机和宿主机通讯
