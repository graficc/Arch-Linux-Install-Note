# Arch + DDE安装

重启进入刚刚安装的 Arch Linux 使用root登录

#### 联网

- 无线

  ```shell
  wifi-menu
  systemctl enable --now dhcpcd
  ```

#### 添加用户

```shell
useradd -m -G wheel -s /bin/bash aaron	#aaron替换为你喜欢的用户名，不能大写^_^
passwd aaron							#设置用户密码
EDITOR=vim visudo
#取消 “# %wheel ALL=(ALL) ALL” 的注释
```

#### 启用32位支持、添加Archlinuxcn源和AUR管理工具

```shell
vim /etc/pacman.conf
# 取消 “#[multilib]”和“#Include = /etc/pacman.d/mirrorlist”注释
echo '[archlinuxcn]' >> /etc/pacman.conf
echo 'Server = https://mirrors.sjtug.sjtu.edu.cn/archlinux-cn/$arch' \
>> /etc/pacman.conf										#这是上海交大的镜像源，可自行替换
pacman -Syy archlinuxcn-keyring							#添加archlinuxcn源的密钥
pacman -S yay											#安装yay，牛批的AUR管理工具
yay --aururl "https://aur.tuna.tsinghua.edu.cn" --save	#对AUR使用清华源的反向代理
```

ps：若添加密钥失败，可进行如下操作

```shell
rm -rf /etc/pacman.d/gnupg
pacman-key --init
pacman-key --populate archlinux
pacman-key --populate archlinuxcn
pacman -Syy archlinuxcn-keyring
```

#### 安装xorg服务

```shell
pacman -S xorg										#xorg服务
pacman -S xf86-video-intel mesa lib32-mesa			#intel核显驱动
pacman -S mesa-libgl lib32-mesa-libgl mesa-vdpau lib32-mesa-vdpau libvdpau-va-gl  #渲染工具
echo "export VDPAU_DRIVER=va_gl" >> /etc/profile	#配置渲染工具
pacman -S xf86-input-libinput						#输入设备驱动
ln -sf /usr/share/X11/xorg.conf.d/40-libinput.conf \
/etc/X11/xorg.conf.d/40-libinput.conf				#初始化配置
```

#### 安装显卡驱动

intel开源驱动已经在上面安装过了，N卡需要再安装闭源驱动

```shell
pacman -S nvidia nvidia-settings nvidia-utils lib32-nvidia-utils libglvnd lib32-virtualgl
```

双显卡笔记本可采用optimus-manager来管理

```shell
pacman -S optimus-manager optimus-manager-qt
```

#### 安装DDE

```shell
pacman -S deepin deepin-extra lightdm lightdm-deepin-greeter
sed -i 's/#greeter-session=example-gtk-gnome/greeter-session=lightdm-deepin-greeter/' \
/etc/lightdm/lightdm.conf							#更改登录界面为deepin
systemctl enable lightdm							#开机自启动lightdm
```

#### 安装网络管理模块

- 网络管理

  ```shell
  pacman -S networkmanager network-manager-applet
  systemctl enable NetworkManager		#注意大小写
  ```

- 蓝牙管理

  ```shell
  pacman -S bluez bluez-utils 
  pacman -S alsa-utils alsa-plugins 
  pacman -S pulseaudio pulseaudio-alsa pulseaudio-bluetooth	#pulseaudio管理模块可选
  systemctl enable bluetooth			#开机自启动蓝牙
  ```

#### 安装中文字体

```shell
pacman -S  adobe-source-han-serif-cn-fonts adobe-source-han-sans-cn-fonts
```

----------------------------------------

**重启进入桌面**

----------------------------------------

#### 更改语言

打开deepin-terminal

```shell
sudo echo 'LANG=zh_CN.UTF-8' > /etc/locale.conf
```

注销重新登录即可

#### 安装中文输入法

打开终端

```shell
sudo pacman -S fcitx fcitx-configtool fcitx-im fcitx-googlepinyin
echo 'export GTK_IM_MODULE=fcitx' > .xprofile
echo 'export QT_IM_MODULE=fcitx' >> .xprofile
echo 'export XMODIFIERS="@im=fcitx"' >> .xprofile
```

重启后生效
