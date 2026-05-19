# archinstall安装系统


## 安装日常软件
```
sudo pacman -S base-devel git curl \
    noto-fonts-cjk noto-fonts-emoji ttf-hack-nerd ttf-roboto \
    firefox firefoxpwa kitty mosh polkit-gnome gnome-keyring

git clone https://aur.archlinux.org/paru.git
cd paru && makepkg -si
paru -S cherry-studio-electron-bin linuxqq wechat clash-verge-rev-bin
paru -S btop tldr tmux kmscon neofetch
```

## zsh配置
```
sudo pacman -S zsh
paru -S autojump-git
sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
cd ~/.oh-my-zsh/plugins
git clone https://github.com/zsh-users/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting
cd ~/.oh-my-zsh/custom/themes
git clone https://github.com/romkatv/powerlevel10k
git -C "${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k" pull
```

## yazi nvim vscode gemini-cli
```
paru -S yazi neovim visual-studio-code-bin
sudo pacman -S luarocks ueberzugpp fd ripgrep ffmpeg 7zip jq poppler fzf zoxide imagemagick chafa 
sudo pacman -S wl-clipboard less bat nodejs npm
```

### 启用vim插件
```
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

### 修复nvim插件错误
```
cd ~/.local/share/nvim/lazy/markdown-preview.nvim
npm install
```

### 应用于root用户
```
sudo cp ~/.zshrc ~/.p10k.zsh ~/.vimrc /root
sudo cp -r ~/.oh-my-zsh/ /root
su root
chsh
```

### gemini-cli
```
#安装nvm以更好管理版本
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.zshrc
nvm install --lts
npm install -g @google/gemini-cli
```

## 系统UI
```
paru -S noctalia-shell \
    cliphist matugen cava wlsunset power-profiles-daemon \
    vicinae-bin
systemctl enable --now --user vicinae.service
```
### gtk主题设置
```
sudo pacman -S nwg-look
paru -S colloid-gtk-theme-git \
    colloid-icon-theme-git colloid-cursors-git
#或者执行安装脚本
cd ~/Public/Builds
git clone https://github.com/vinceliuice/Colloid-gtk-theme.git
cd Colloid-gtk-theme
#在~/.config/niri/config.kdl中添加
cursor {
    xcursor-theme "/usr/share/icons/Colloid-dark-cursors"
    xcursor-size 24
}
```

### 主题
```
paru -S colloid-gtk-theme-git colloid-icon-theme-git colloid-cursors-git
#~/.config/niri/config.kdl加入如下
cursor {
    xcursor-theme "/usr/share/icons/Colloid-dark-cursors"
    xcursor-size 24
}
```

## 文件管理器配置

### nemo本体和右键功能
```
sudo pacman -S nemo cinnamon-translations nemo-fileroller nemo-preview \
    p7zip unrar unzip zip file-roller lz4 zstd
#右键终端打开使用kitty
gsettings set org.cinnamon.desktop.default-applications.terminal exec 'kitty'
```

### 设备管理与虚拟文件系统
```
sudo pacman -S udisks2 \
    gvfs gvfs-afc gvfs-gphoto2 gvfs-mtp gvfs-goa
```

### 文件系统支持
```
sudo pacman -S ntfs-3g exfatprogs f2fs-tools dosfstools 
```

### 加密与凭据管理
```
sudo pacman -S gnome-keyring gcr
```

### nfs与smb访问
```
sudo pacman -S gvfs-nfs gvfs-smb smbclient nss-mdns avahi
sudo systemctl enable --now avahi-daemon.service
```

### 替换掉nautilus
```
sudo pacman -S xdg-desktop-portal-gtk xdg-desktop-portal-wlr
sudo pacman -Rdd xdg-desktop-portal-gnome
sudo pacman -Rss nautilus
xdg-mime default nemo.desktop inode/directory
mkdir -p ~/.config/xdg-desktop-portal/
vim ~/.config/xdg-desktop-portal/portals.conf

[preferred]
# 默认使用 gtk 后端处理所有请求
default=gtk
# 文件夹打开请求明确指向 gtk (它会调用 xdg-open -> nemo)
org.freedesktop.impl.portal.FileChooser=gtk
# 如果你用 niri，截图和录屏建议用 wlr (需要安装 xdg-desktop-portal-wlr)
org.freedesktop.impl.portal.ScreenCast=wlr
org.freedesktop.impl.portal.Screenshot=wlr
```


## 中文本地化

### 安装中文输入法
```
paru -S fcitx5-im fcitx5-rime rime-ice \
    fcitx5-nord-pink \
    rime-pinyin-moegirl rime-pinyin-moegirl
```

### 添加词典
```
cp /usr/share/rime-data/rime_ice.dict.yaml ~/.local/share/fcitx5/rime/rime_ice.dict.yaml
vim ~/.local/share/fcitx5/rime/rime_ice.dict.yaml

import_tables:
  ...
  ...
  - zhwiki
  - moegirl
```

### 设置雾凇拼音
```
mkdir ~/.local/share/fcitx5/rime
vim ~/.local/share/fcitx5/rime/default.custom.yaml

patch:
  # 仅使用「雾凇拼音」的默认配置，配置此行即可
  __include: rime_ice_suggestion:/
  # 以下根据自己所需自行定义
  __patch:
    menu/page_size: 8   #候选词个数
```

### fcitx5-nord-pink配色优化
```
#ff7fa1
#ff236c
#ffdbe6
#ff236c
```

## office
```
sudo pacman -S libreoffice-still libreoffice-still-zh-cn
```

## 多媒体
```
paru -S imv mpv ani-cli
```

## N卡闪屏问题
```
#sudo nano /etc/default/grub
#sudo grub-mkconfig -o /boot/grub/grub.cfg
#修改 GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet nvidia.NVreg_PowerMizerMode=1"
```

## 系统引导

### 多系统引导
```
sudo pacman -S os-prober
sudo -e /etc/default/grub

GRUB_DEFAULT=saved
GRUB_TIMEOUT=10
GRUB_SAVEDEFAULT=true

sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### grub 主题
```
paru -S grub-theme-lain
git clone https://github.com/uiriansan/LainGrubTheme.git
cd LainGrubTheme
sudo ./install.sh
sudo ./patch_entries.sh
sudo grub-mkconfig -o /boot/grub/grub.cfg

#调整字体大小
sudo grub-mkfont --output=/boot/grub/fonts/hack_bold_big.pf2 --size=48 "/usr/share/fonts/TTF/HackNerdFont-Bold.ttf"
sudo -e /etc/default/grub

GRUB_FONT="/boot/grub/fonts/hack_bold_big.pf2"
```

### sddm 主题
```
paru -S sddm-lain-wired-theme
sudo mkdir /etc/sddm.conf.d
sudo cp -p /usr/lib/sddm/sddm.conf.d/default.conf /etc/sddm.conf.d/sddm.conf
sudo -e /etc/sddm.conf.d/sddm.conf

GreeterEnvironment=QT_SCREEN_SCALE_FACTORS=2,QT_FONT_DPI=192
Current=sddm-lain-wired-theme
```

### grub 安全启动
```
paru -S shim-signed
sudo pacman -S sbsigntools mokutil
```

## chromebook声卡
```
git clone https://github.com/WeirdTreeThing/chromebook-linux-audio.git
cd chromebook-linux-audio
./setup-audio
sudo pacman -S pavucontrol
```

## linux-surface

### linux-surface内核
```
https://github.com/linux-surface/linux-surface/wiki/Installation-and-Setup#Arch

curl -s https://raw.githubusercontent.com/linux-surface/linux-surface/master/pkg/keys/surface.asc \
    | sudo pacman-key --add -

sudo pacman-key --finger 56C464BAAC421453
sudo pacman-key --lsign-key 56C464BAAC421453

sudo -e /etc/pacman.conf
[linux-surface]
Server = https://pkg.surfacelinux.com/arch/

sudo pacman -Syu

sudo pacman -S linux-surface linux-surface-headers iptsd
```

### surface的安全启动
