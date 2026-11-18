# archinstall安装系统

## 基本配置
```bash
sudo pacman -S fcitx5-im fcitx5-rime firefox firefoxpwa kitty kmscon \
noto-fonts-emoji noto-fonts-cjk ttf-hack-nerd ttf-roboto \
base-devel git curl mosh \
bat btop less tmux fastfetch \
polkit-gnome gnome-keyring tldr

# paru
git clone https://aur.archlinux.org/paru.git
cd paru && makepkg -si
paru -S clash-verge-rev-bin yazi neovim \
luarocks ueberzugpp fd ripgrep ffmpeg 7zip jq poppler fzf zoxide imagemagick chafa wl-clipboard
```


## CMD

### git
```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
git config --global init.defaultBranch main
ssh -T -p 443 git@ssh.github.com
git config core.sshCommand "ssh -o Hostname=ssh.github.com -p 443"
```


### oh-my-zsh
```bash
sudo pacman -S zsh
paru -S autojump-git

sh -c "$(wget https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"

cd ~/.oh-my-zsh/custom/themes
git clone --depth 1 https://github.com/romkatv/powerlevel10k
```
```bash
# 使用国内源
git clone --depth 1 https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git
cd ohmyzsh/tools
REMOTE=https://mirrors.tuna.tsinghua.edu.cn/git/ohmyzsh.git sh install.sh

cd ~/.oh-my-zsh/custom/themes
git clone --depth 1 https://gitee.com/romkatv/powerlevel10k.git
```

### nodejs npm nvm
```bash
sudo pacman -S nodejs npm

curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.zshrc

nvm install --lts
```

### ide and coding agent
```bash
paru -S visual-studio-code-bin curosr-bin

npm install -g @openai/codex
npm install -g opencode-ai@latest
npm install -g @anthropic-ai/claude-code
```

```
# 启用vim-plug
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim

# 修复nvim插件错误
cd ~/.local/share/nvim/lazy/markdown-preview.nvim
npm install
```


## system settings

### UI
```bash
paru -S noctalia \
cliphist matugen cava wlsunset power-profiles-daemon

sudo pacman -S nwg-look
paru -S colloid-gtk-theme-git colloid-icon-theme-git colloid-cursors-git
```

### file manager

#### thunar本体和基本工具
```bash
sudo pacman -S thunar thunar-archive-plugin thunar-volman \
tumbler ffmpegthumbnailer exo \
p7zip unrar unzip zip file-roller lz4 zstd
```

#### 支持
```bash
# 设备管理与虚拟文件系统
sudo pacman -S udisks2 \
gvfs gvfs-afc gvfs-gphoto2 gvfs-mtp gvfs-goa

# 文件系统支持
sudo pacman -S ntfs-3g exfatprogs f2fs-tools dosfstools 

# 加密与凭据管理
sudo pacman -S gnome-keyring gcr

# nfs与smb访问
sudo pacman -S gvfs-nfs gvfs-smb smbclient nss-mdns avahi
sudo systemctl enable --now avahi-daemon.service
```

#### 移除nautilus/nemo
```bash
sudo pacman -Rss nautilus
rm -rf ~/.config/nautilus

sudo pacman -Rsn nemo nemo-fileroller nemo-preview \
cinnamon-translations cinnamon-desktop xapp exempi
rm -rf ~/.config/nemo
```

#### 设置为默认软件
```bash
sudo pacman -Rdd xdg-desktop-portal-gnome
sudo pacman -S xdg-desktop-portal-gtk 
#sudo pacman -S xdg-desktop-portal-wlr

xdg-mime default thunar.desktop inode/directory
xdg-mime default firefox.desktop x-scheme-handler/http
xdg-mime default firefox.desktop x-scheme-handler/https
xdg-mime default firefox.desktop text/html
xdg-settings set default-web-browser firefox.desktop
```
配置：`~/.config/xfce4/helpers.rc`
```ini
TerminalEmulator=kitty
FileManager=thunar
WebBrowser=firefox
```
```
mkdir -p ~/.config/xdg-desktop-portal
cat > ~/.config/xdg-desktop-portal/niri-portals.conf << 'EOF'
[preferred]
default=gtk
org.freedesktop.impl.portal.FileChooser=gtk
org.freedesktop.impl.portal.ScreenCast=wlr
org.freedesktop.impl.portal.Screenshot=wlr
EOF
```

配置：`~/.config/xfce4/helpers.rc`
```ini
TerminalEmulator=kitty
FileManager=thunar
WebBrowser=firefox
```


## 日常软件工具

### screenshot
```bash
sudo pacman -S grim slurp wl-clipboard xdg-desktop-portal xdg-desktop-portal-gtk xdg-desktop-portal-wlr
systemctl --user import-environment WAYLAND_DISPLAY XDG_CURRENT_DESKTOP
systemctl --user restart xdg-desktop-portal
```

配置：`config.kdl`
```
spawn-sh-at-startup "dbus-update-activation-environment --systemd WAYLAND_DISPLAY XDG_CURRENT_DESKTOP=niri"
Mod+S { spawn "sh" "-c" "grim -g \"$(slurp)\" - | wl-copy"; }
```

### linuxqq and wechat
```bash
sudo pacman -S flatpak
flatpak install flathub com.qq.QQ
flatpak install flathub com.tencent.WeChat
flatpak install flathub com.github.tchx84.Flatseal
flatpak override --user \
  --env=GTK_IM_MODULE=fcitx \
  --env=QT_IM_MODULE=fcitx \
  --env=XMODIFIERS=@im=fcitx \
  --env=QT_QPA_PLATFORM=xcb \
  com.tencent.WeChat
```

### clash-for-linux-install
```bash
git clone --branch master --depth 1 https://gh-proxy.org/https://github.com/nelvko/clash-for-linux-install.git \
  && cd clash-for-linux-install \
  && bash install.sh
clashon && clashtun on
```

### adb
```bash
sudo pacman -S android-tools android-udev
sudo usermod -aG adbusers $USER
sudo udevadm control --reload-rules && sudo systemctl restart systemd-udevd.service
```


## 中文本地化

### 安装中文输入法
```bash
paru -S fcitx5-im fcitx5-rime rime-ice \
    fcitx5-nord-pink \
    rime-pinyin-moegirl rime-pinyin-moegirl
```

### 添加词典
```bash
cp /usr/share/rime-data/rime_ice.dict.yaml ~/.local/share/fcitx5/rime/rime_ice.dict.yaml
```

配置：`~/.local/share/fcitx5/rime/rime_ice.dict.yaml`
```
import_tables:
  ...
  ...
  - zhwiki
  - moegirl
```

### 设置雾凇拼音
```bash
mkdir ~/.local/share/fcitx5/rime
```

配置：`#~/.local/share/fcitx5/rime/default.custom.yaml`
```
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


## 多媒体
```bash
paru -S imv mpv vlc ani-cli
```


## N卡闪屏问题
```bash
#sudo nano /etc/default/grub
#sudo grub-mkconfig -o /boot/grub/grub.cfg
#修改 GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet nvidia.NVreg_PowerMizerMode=1"
```


## bootloader

### os-prober
```
sudo pacman -S os-prober
```

### grub prefix
```
GRUB_DEFAULT=saved
GRUB_TIMEOUT=10
GRUB_SAVEDEFAULT=true
GRUB_DISABLE_OS_PROBER=false
```

### grub.cfg
```
sudo ln -s /boot/efi/grub /boot/grub
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### grub 主题
```bash
git clone https://github.com/uiriansan/LainGrubTheme.git
cd LainGrubTheme
sudo ./install.sh
sudo ./patch_entries.sh
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### sddm 主题
```bash
paru -S sddm-lain-wired-theme
sudo mkdir /etc/sddm.conf.d
sudo cp -p /usr/lib/sddm/sddm.conf.d/default.conf /etc/sddm.conf.d/sddm.conf
```

### /etc/sddm.conf.d/sddm.conf
```
GreeterEnvironment=QT_SCREEN_SCALE_FACTORS=2,QT_FONT_DPI=192
Current=sddm-lain-wired-theme
```

### grub 安全启动
```bash
paru -S shim-signed
sudo pacman -S sbsigntools mokutil
```
