# Nemo → Thunar 迁移 / 全新安装

## 1. 安装 Thunar 生态
```bash
sudo pacman -S thunar thunar-archive-plugin thunar-volman \
    tumbler ffmpegthumbnailer exo
```

## 7. 设置右键终端为 kitty
```bash
exo-preferred-applications
# GUI 中设 TerminalEmulator=kitty
```
或直接写配置文件：
```bash
mkdir -p ~/.config/xfce4
cat > ~/.config/xfce4/helpers.rc << 'EOF'
TerminalEmulator=kitty
EOF
```

## 8. 设 Thunar 为默认文件管理器
```bash
xdg-mime default thunar.desktop inode/directory
```

验证：
```bash
xdg-mime query default inode/directory
# → thunar.desktop
```

## 9. portal 配置（与 niri 集成）
```bash
mkdir -p ~/.config/xdg-desktop-portal
cat > ~/.config/xdg-desktop-portal/portals.conf << 'EOF'
[preferred]
default=gtk
org.freedesktop.impl.portal.FileChooser=gtk
org.freedesktop.impl.portal.ScreenCast=wlr
org.freedesktop.impl.portal.Screenshot=wlr
EOF
```

## 11. 替换掉 nautilus（如有）
```bash
sudo pacman -S xdg-desktop-portal-gtk xdg-desktop-portal-wlr
sudo pacman -Rdd xdg-desktop-portal-gnome
sudo pacman -Rss nautilus
```

## 12. 移除 nemo 残留（仅迁移时执行）
```bash
sudo pacman -Rsn nemo nemo-fileroller nemo-preview \
    cinnamon-translations cinnamon-desktop xapp exempi
rm -rf ~/.config/nemo
```
