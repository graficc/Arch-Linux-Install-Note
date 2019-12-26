# Arch + KDE安装

重启进入刚刚安装的 Arch Linux 使用root登录

#### 联网

- 有线和USB网络共享

  ```sh
  systemctl enable --now dhcpcd
  ```

- 无线

  ```sh
  wifi-menu
  systemctl enable --now dhcpcd
  ```

#### 添加用户

```sh
useradd -m -G wheel -s /bin/bash aaron    # aaron替换为你喜欢的用户名，不能大写^_^
passwd aaron    # 设置用户密码
EDITOR=vim visudo
# 取消 “# %wheel ALL=(ALL) ALL” 的注释
```

#### 启用32位支持、添加Archlinuxcn源和AUR管理工具

```sh
vim /etc/pacman.conf
# 取消 “#[multilib]” 和 “#Include = /etc/pacman.d/mirrorlist”注释
echo '[archlinuxcn]' >> /etc/pacman.conf
echo 'Server = https://mirrors.sjtug.sjtu.edu.cn/archlinux-cn/$arch' \
>> /etc/pacman.conf    # 这是上海交大的镜像源，可自行替换
pacman -Syy archlinuxcn-keyring    # 添加archlinuxcn源的密钥
```

ps：若添加密钥失败，可进行如下操作

```sh
rm -rf /etc/pacman.d/gnupg
pacman-key --init
pacman-key --populate archlinux
pacman-key --populate archlinuxcn
pacman -Syy archlinuxcn-keyring
```

#### 安装xorg服务

```sh
pacman -S xorg    # xorg服务
pacman -S xf86-video-intel mesa lib32-mesa    # intel核显驱动
pacman -S xf86-input-libinput    # 输入设备驱动
pacman -S xf86-input-synaptics    # 触摸板驱动
ln -sf /usr/share/X11/xorg.conf.d/40-libinput.conf \
/etc/X11/xorg.conf.d/40-libinput.conf    # 初始化输入设备配置
ln -sf /usr/share/X11/xorg.conf.d/70-synaptics.conf \
/etc/X11/xorg.conf.d/70-synaptics.conf    # 初始化触摸板配置
```

#### 安装显卡驱动

intel开源驱动已经在上面安装过了，N卡需要再安装闭源驱动

```sh
pacman -S nvidia-lts nvidia-settings nvidia-utils lib32-nvidia-utils
```

双显卡笔记本可采用optimus-manager来管理

```sh
pacman -S optimus-manager optimus-manager-qt
```

#### 安装KDE

```sh
pacman -S plasma-meta kdebase-meta kdegraphics-meta    # 安装kde桌面和部分工具
pacman -S sddm sddm-kcm    # 安装kde登录界面
systemctl enable sddm    #设置登录界面自启动
```

#### 安装网络管理模块

- 网络管理

  ```sh
  pacman -S networkmanager
  systemctl enable NetworkManager    # 设置网络管理模块自启动 注意大小写
  ```

- 蓝牙管理

  ```sh
  pacman -S bluez bluez-utils
  systemctl enable bluetooth    # 设置蓝牙模块开机自启动
  ```

#### 安装中文字体

```sh
pacman -S adobe-source-han-serif-cn-fonts adobe-source-han-sans-cn-fonts    # 思源系列字体
```

----------------------------------------

**重启进入桌面**

----------------------------------------

#### 更改语言

打开konsole

```sh
sudo echo 'LANG=zh_CN.UTF-8' > /etc/locale.conf
```

注销重新登录即可

#### 安装中文输入法

打开konsole

```sh
sudo pacman -S fcitx fcitx-configtool fcitx-im fcitx-googlepinyin kcm-fcitx    # 安装输入法
echo 'export GTK_IM_MODULE=fcitx' > .xprofile
echo 'export QT_IM_MODULE=fcitx' >> .xprofile
echo 'export XMODIFIERS="@im=fcitx"' >> .xprofile
```

重启后生效

#### 安装TLP笔记本电源管理系统

打开konsole

```sh
sudo pacman -S tlp tlp-rdw ethtool smartmontools    # 安装TLP
sudo systemctl enable tlp.service
sudo systemctl enable tlp-sleep.service    # 设置TLP自启动
sudo systemctl mask systemd-rfkill.service
sudo systemctl mask systemd-rfkill.socket    # 屏蔽部分服务以免冲突
```

重启后生效

