**主要参考了 Arch Wiki 的官方安装指南、借助了B站UP主[uid=9202840] 的思路和视频讲解并结合了自己实际安装操作遇到的问题和解决过程，主要自用**

# 安装
## 开始
进入Live环境后先连接网络，连无线网则使用iwd提供的命令行工具，输入
```bash
iwctl
device list  #列出设备
station 设备名 scan  #扫描网络
station 设备名 get-networks
station 设备名 connect 无线网名  #wifi名不能是中文
exit  #退出iwctl
```
使用`ping`测试网络
使用`timedatectl`确认是否开启了NTP并同步到了UTC
可使用`timedatectl set-ntp true`手动开启NTP
接下来配置镜像源：
`reflector -a  12 -c cn -f 10 --sort rate --v --save /etc/pacman.d/mirrorlist`
更新数据库并安装密钥：
`pacman -Sy archlinux-keyring`
可安装yazi方便浏览文件

## 硬盘分区
**处理硬盘分区**
存在`lsblk`、`fdisk`、`cfdisk`工具用来管理磁盘
若第一次使用该硬盘（或其中某个分区）则选用gpt模式
准备至少100MB（空间充裕的话可以准备多点）作为ESP，将剩余空间处理为Linux filesystem类型
- **双系统需要注意：**
未避免Windows弄坏Linux的引导文件，建议为两者准备各自独立的EFI分区
- **单硬盘双系统需要注意：**
如nvme0n1号硬盘同时装载了Windows，应当直接将目标free space分别划分成nvme0n1px和nvme0n1py号分区分别供给EFI和Linux文件系统，而不应该将其全部划成nvme0n1px号后又进一步细分为nvme0n1pxp1、nvme0n1pxp2号分区，因为二级结构是无法被识别到的

## 挂载ESP（EFI System Partition)**
通常挂载到/boot,但/boot已经存放体积较大的内核文件，因此不建议再挂载到/boot,可以挂载到/boot/efi或者/efi，ArchWiki推荐/efi
并且为了实现Arch的快照功能，根分区需要处理成BTRFS文件系统，ESP必须是FAT类，而BTRFS不能快照作为FAT文件系统的ESP
输入以下命令将两个分区分别格式化为FAT32和BTRFS
```bash
mkfs.fat -F 32 /dev/ESP号
mkfs.btrfs /dev/LFP（Linux文件系统分区）号
```
### 创建子卷
用于设置快照的范围，将系统数据和用户数据分开快照
输入以下命令用于创建同级子卷
```bash
mount -t btrfs /dev/LFP /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
```
由于输入了`mount -t btrfs /dev/LFP /mnt`使得根分区已经挂载到了/mnt，为了将root子卷挂载到/mnt，需要先取消挂载，然后将root子卷挂载到/mnt：
```bash
umount /mnt
mount -t btrfs -o subvol=/@,compress=zstd /dev/LFP /mnt
mount --mkdir -t btrfs -o subvol=/@home,compress=zstd /dev/LFP /mnt/home
```
- 其中使用了compress=zstd来使用zstd压缩算法来进行透明压缩，可以有益于硬盘，等级默认是3,可以使用`zstd=数字`来指定等级（1-15）

### 挂载ESP
`mount --mkdir /dev/ESP /mnt/efi`

## 安装系统
### 安装系统文件
使用如下命令安装系统
`pacstrap -K /mnt base base-devel linux linux-firmware btrfs-progs`
- 也可以安装linux-zen性能特调内核（本人使用该内核进入系统会出现死机现象，原因暂未查明，目前猜测可能是未安装显卡驱动引起的驱动冲突导致的）plus:安装显卡驱动后可以通过zen内核进入系统，但无法识别到外接显示器，只能使用笔记本内置屏幕，原因暂未查明

`pacstrap -K /mnt base base-devel linux-zen linux-firmware btrfs-progs`
如果是marvell网卡则需要额外安装marvell固件包
`pacstrap -K /mnt base base-devel linux linux-firmware btrfs-progs linux-firmware-marvell`
### 安装基本功能性软件
安装联网工具，文本编辑器，权限管理器，CPU优化、修复工具
`pacstrap /mnt networkmanager vim sudo intel-ucode`
amd用户就用`amd-ucode`，想用终端联网就装`iwd`(**iwd可能需要配dhcpcd使用**）
### 自动挂载文件系统
输入以下命令生成fstab文件以使系统自动完成系统文件挂载
`genfstab -U /mnt > /mnt/etc/fstab`

## chrange root
###进入arch-chroot
`arch-chroot /mnt`
### 设置时区
```bash
timedatectl set-timezone Asia/Shanghai
--systohc  #在/etc下生成了adjtime文件用来调整时间误差
```
### 设置本地化
`vim /etc/locale.gen`
取消`en_US.UTF-8 UTF-8`和`zh_CN.UTF-8 UTF-8`两行的注释
生成本地化文件：
`locale-gen`
设置本地化：
`vim /etc/locale.conf`
`vim .config/locale.conf`
都写入`LANG=en_US.UTF-8`，用来设置全局本地使用英文，避免在使用root登陆tty时终端文字变成方块以致无法识别
设置所有新建用户使用语言
```bash
mkdir /etc/skel/.config
vim /etc/skel/.config/locale.conf
```
写入`LANG=zh_CN.UTF-8`
### 主机
`vim/etc/hostname`
写入自己想要的主机名字
`passwd root`
设置root用户密码
### 引导加载程序和双系统
```bash
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/efi ==boot-directory=/efi ==bootloader-id=(不填则默认为arch)
ln -s /efi/grub /boot/grub  #在grub的默认位置创造链接指向/efi/grub，因为多数软件都会默认grub装在了/boot
grub-mkconfig -o /boot/grub/grub.cfg  **#生成grub配置文件，每次手动编辑过grub都要手动生成一次以让修改生效**
```
- 双系统
`pacman -S os-prober exfat-utils`
os-prober用来搜索其他系统，exfat-utils能找到Windows的EFI分区
(也可以通过挂载Windows的EFI分区的方式来找到Windows)
`vim /etc/default/grub`
取消`GRUB_DISABLE_OS_PROBER=false`的注释以允许grub生成配置文件时使用os-prober搜索其他系统
取消`GRUB_SAVEDEFAULT=true`的注释以允许grub获得启动项记忆功能
将`GRUB_DEFAULT=0`修改为`GRUB_DEFAULT=saved`
`GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet"`中，把`loglevel`改成5,并删除`quiet`
- Zram内存压缩和交换空间
可以提升系统运行速度，增加可用内存
```bash
pacman -S zram-generator
vim /etc/systemd/zram/zram-generator.conf
```
写入：
```bash
[zram0]
zram-size=ram
compression-algorithm = zstd
```
`vim /etc/default/grub`
在`GRUB_CMDLINE_LINUX_DEFAULT`参数里加上`zswap.enabled=0`

### 进入系统
`reboot`重启，通过grub进入Arch
输入`root`进入root用户
```bash
systemctl enable --now NetworkManager
nmtui
```
连接wifi
```bash
pacman -S fastfetch cmatrix
fastfetch  #不fetch的Arch是没有灵魂的
cmatrix  #可以欣赏代码雨享受一下
```
## 一些基本设置
```bash
pacman -Syu
vim /etc/environment
```
追加写入`EDITOR=vim`来设置系统默认文本编辑器为vim
### 创建用户
```bash
useradd -mG wheel 用户名  #-mG分别使创建用户时创建home目录，并将该用户添加进wheel组-拥有管理员权限的组
passwd 用户名
visudo
```
取消`%wheel ALL=(ALL:ALL) ALL`行的注释
### 开启32位源
`vim /etc/pacman.conf`
取消
```bash
[multilib]
Include = /etc/pacman.d/mirrorlist
```
两行的注释，并在末尾追加一行写入
```bash
[archlinuxcn]
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
Server = https://mirrors.hit.edu.cn/archlinuxcn/$arch
Server = https://repo.huaweicloud.com/archlinuxcn/$arch
```
然后运行该命令同步数据并安装Archlinuxcn的密钥
`pacman -Sy archlinuxcn-keyring`
### 下载AUR助手
有`yay`和`paru`两种供选择
`pacman -S yay paru`
- 某些扩展较多的软件可以用flatpak安装，通常比AUR上的更好用
```
sudo pacman -S flatpak
sudo flatpak remote-modify flathub --url=https://mirror.sjtu.edu.cn/flathub
```


**♥️♥️现在已经完成了Arch的安装和一些基本配置♥️♥️**

# 进阶配置
## 更换shell
普通的bash比较低效，可以更换一个shell来提高效率，但不建议更换root用户的bash，仅让普通用户使用其他shell
```
sudo pacman -S fish
chsh -s /usr/bin/fish
```
`fish`和`bash`都存放在`/usr/bin/`目录下
重启以生效

## 快照（snapshot）
```bash
pacman -S snapper snap-pac btrfs-assistant grub-btrfs inotify-tools
systemctl enable --now grub-btrfsd  #开启快照启动项的systemd服务
```
重启一次电脑使snap-pac自动在pacman的时候创建快照

登录root
```bash
snapper -c root create-config /
snapper -c home create-config /home
snapper -c root create --description "helloworld"  #可以创建名为“helloworld”的root快照
pacman -S linux-lts  #lts内核不会频繁更新，系统出现异常时在快照回档之前可以尝试用LTS内核进入系统排查是否是内核问题引起的系统异常
grub-mkconfig -o /boot/grub/grub.cfg
```

## 代理翻墙
- 这一步位于安装桌面环境后操作
使用AUR安装v2rayN的Linux GUI工具：
`yay -S v2rayn-bin`
也可以下载zip包自己编译运行：
`yay -S v2rayn`
运行需要依赖.NET运行时环境，运行出错请确保安装了`dotnet-runtime`和`dotnet-sdk`
- 创建链接
`sudo ln -s /opt/v2rayn-bin/v2rayN /usr/local/bin/v2rayN`
使得可以在终端直接输入`v2rayN`以启动v2rayN

## 安装英伟达显卡驱动
具体流程：去ArchLinux的英伟达驱动官网找到自己显卡型号对应的系，安装需要的、对应的包
我5060显卡，因此需要安装blackwell系列的nvidia-open-dkms，并安装对应的头文件
```bash
pacman -S --need linux-headers linux-lts-headers
pacman -S nvidia-open-dkms
pacman -S nvidia-utils lib32-nvidia-utils nvidia-settings  #库、工具集和英伟达服务器设置
```
- 安装过程中遭遇了一个叫nouveau的驱动占用了显卡，导致新安装的显卡驱动无法生效，需要将nouveau驱动禁用：
`sudo vim /etc/modprobe.d/blacklist.conf`
写入`blacklist nouveau`将nouveau拉入黑名单
然后输入：
`sudo mkinitcpio -P`
重新生成所有已安装内核的初始内存盘
然后重启系统，输入：
`lsmod | grep nouveau`
没有内容输出及代表nouveau已被禁用
再输入：
`nvidia-smi`
输入信息显示显卡正在工作的信息即完成

## 视频编解码的驱动安装
根据官网Hardware video acceleration找到自己需要的包
由于已经禁用了Nouveau，安装`nvidia-utils`&`libva-nvidia-dirver`
重启电脑生效
**为了让Firefox使用N卡编解码，按照`nvidia-vaapi-driver`仓库的教程进行设置**

## 音视频服务和蓝牙
`sudo pacman -S sof-firmware alsa-ucm-conf alsa-firmware`
**pipewire和蓝牙**
`sudo pacman -S pipewire wireplumber pipewire-pulse pipewire-alsa pipewire-jack bluez blueman`
**启用服务**
```
systemctl --user enable pipewire wireplumber pipewire-pulse
sudo systemctl enable --now bluetooth
```
**性能模式切换工具**
```
sudo pacman -S power-profiles-daemon
sudo systemctl enable --now power-profiles-daemon
```