# Arch Linux 安装

#### 安装前的准备

将archlinux的iso烧录到U盘

```shell
sudo umount /dev/sdX*               #卸载U盘
sudo dd oflag=sync status=progress bs=4M \
if=./archlinux.iso of=/dev/sdX    #烧录iso
```

#### 启动到live环境

插入U盘，启动到archlinux的live环境

ps：N卡最好在启动时按下e并在末尾加入 "modprobe.blacklist=nouveau" 以禁用 nouveau 开源驱动

#### 验证启动模式

```shell
ls /sys/firmware/efi/efivars
```

如果有结果，系统就是以UEFI启动的，否则是BIOS模式启动的，本文只讲解UEFI+GPT

#### 联网

- 有线

  插了网线的的启动是就会用dhcpcd获取有线的ip地址，也可以ping一下百度测试一下

- 无线

  ```shell
  wifi-menu                                  #连接wifi
  dhcpcd                                        #获取ip地址
  ping -c4 www.baidu.com    #测试连接
  ```

- USB网络共享

  ```shell
  dhcpcd                                        #即可自动获取ip地址
  ```

#### 更换软件源

```shell
vim /etc/pacman.d/mirorrlist
```

在第一行加入镜像源，这个是东软的，可自行更换 

Server = <https://mirrors.neusoft.edu.cn/archlinux/$repo/os/$arch>

#### 更新系统时间

```shell
timedatectl set-ntp true
timedatectl status                            #可选，查看系统现在的时间状态
```

#### 分区

- 我的分区方案：

| 挂载点    | 分区           | 分区类型                   | 大小  | 文件系统 |
| --------- | -------------- | -------------------------- | ----- | -------- |
| /boot/efi | /dev/nvme0n1p1 | EFI系统分区（ef00）        | 512MB | fat32    |
| /         | /dev/nvme0n1p2 | Linux 根目录（8300）       | 50GB  | xfs      |
| /home     | /dev/nvme0n1p3 | 个人数据目录（8300）       | 200GB | xfs      |
| [SWAP]    | /dev/nvme0n1p4 | Linux swap交换分区（8200） | 4GB   | swap     |

​    GPT分区表最好使用 gdisk 命令或者 cgdisk 交互命令

```shell
gdisk /dev/nvme0n1                       #更换为自己想要安装到的硬盘
```

​    分完区可以用 lsblk 命令检查一下

- 格式化和挂载分区

  ```sh
  mkfs.fat -F32 /dev/nvme0n1p1                         #格式化efi分区
  mkfs.xfs /dev/nvme0n1p2                                  #格式化根目录分区
  mkfs.xfs /dev/nvme0n1p3                                  #格式化home分区
  mkswap /dev/nvme0n1p4                                  #格式化swap分区
  swapon /dev/nvme0n1p4                                   #启用swap
  mount /dev/nvme0n1p2 /mnt                          #把根分区挂载到/mnt
  mkdir -p /mnt/boot/efi /mnt/home               #建立/boot/efi和/home目录
  mount /dev/nvme0n1p1 /mnt/boot/efi       #挂载efi分区到/boot/efi
  mount /dev/nvme0n1p3 /mnt/home           #挂载home分区到/home
  ```

#### 开始安装

```shell
pacstrap /mnt base base-devel linux-lts linux-lts-headers linux-firmware dosfstools xfsprogs libmtp mtpfs net-tools dhcpcd dialog iw netctl wireless_tools wpa_supplicant inetutils vim man-db man-pages bash-completion
```

base组更改之后需要加装很多东西，管理文件系统的、联网的、文本编辑的......

#### 配置系统

- Fstab

  生成fstab文件

  ```shell
  genfstab -U /mnt >> /mnt/etc/fstab
  ```

  建议用vim查看一下生成后的fstab

- Chroot

  Change root 到刚刚安装的系统

  ```shell
  arch-chroot /mnt
  ```

- 本地化

  - 时区

    ```shell
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime    #更改时区
    hwclock --systohc                                                                                 #应用到硬件时间
    ```

  - 语言

    ```shell
    vim /etc/locale.gen
    ```

    移除下列语言的注释（#）即可

    ```shell
    en_US.UTF-8 UTF-8
    zh_CN.UTF-8 UTF-8
    ```
    

生成locale信息
    
```shell
locale-gen
```

更改语言环境为英文，避免乱码
    
```shell
echo 'LANG=en_US.UTF-8' > /etc/locale.conf
```

- 网络

  ```shell
  echo 'OMEN-ARCH' > /etc/hostname    #更改OMEN-ARCH为你喜欢的主机名
  vim /etc/hosts
  ```

  加入以下内容

  ```shell
  127.0.0.1        localhost
  ::1              localhost
  127.0.1.1        OMEN-ARCH.localdomain    OMEN-ARCH
  ```

- Initramfs

  ```sh
  pacman -Syy intel-ucode             #安装intel微码
  mkinitcpio -P
  ```
  
- 设定Root密码

  ```shell
  passwd
  ```

#### 安装引导

```shell
pacman -S grub efibootmgr os-prober
grub-install --target=x86_64-efi --efi-directory=/boot/efi \
--bootloader-id=GRUB_ARCH --recheck    #安装grub引导
os-prober                                                                #探测其他操作系统，注意需将其他系统的efi分区挂载
grub-mkconfig -o /boot/grub/grub.cfg      #生成grub配置    
```

N卡的电脑建议添加禁用 nouveau 的内核参数到grub，方法如下

```shell
vim /etc/default/grub    
# 在GRUB_CMDLINE_LINUX_DEFAULT="" 添加 "modprobe.blacklist=nouveau"
grub-mkconfig -o /boot/grub/grub.cfg      #重新生成grub配置
```

- 重启

  ```shell
  exit
  umount -a
  reboot
  ```
