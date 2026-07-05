# Linux 默认应用机制总纲

## 1. xdg-mime — 文件类型/协议关联（核心）

管 `xdg-open` 的行为：双击文件夹、点链接、打开文件。

```bash
# 查询当前默认
xdg-mime query default inode/directory
xdg-mime query default x-scheme-handler/http
xdg-mime query default text/html
xdg-mime query default application/pdf

# 设置
xdg-mime default thunar.desktop inode/directory
xdg-mime default firefox.desktop x-scheme-handler/http
xdg-mime default firefox.desktop x-scheme-handler/https
xdg-mime default firefox.desktop text/html
```

配置文件：`~/.config/mimeapps.list`

```ini
[Default Applications]
inode/directory=thunar.desktop
x-scheme-handler/http=firefox.desktop
x-scheme-handler/https=firefox.desktop
text/html=firefox.desktop
```

---

## 2. xdg-settings — 网页浏览器快捷设置

`xdg-mime` 的语法糖，只封装了默认浏览器：

```bash
xdg-settings set default-web-browser firefox.desktop
xdg-settings get default-web-browser
# → firefox.desktop
```

本质写入 `mimeapps.list` 的 `x-scheme-handler/http` 和 `https`。

---

## 3. exo helpers — Thunar 右键终端

Thunar "Open Terminal Here" → `exo-open --launch TerminalEmulator` → 查配置文件。

配置：`~/.config/xfce4/helpers.rc`

```ini
TerminalEmulator=kitty
```

也管文件管理器和浏览器回退：

```ini
TerminalEmulator=kitty
FileManager=thunar
WebBrowser=firefox
```

GUI 工具：**无（`exo-preferred-applications` 已被 XFCE 废弃，无有效 GUI）**

---

## 4. xdg-desktop-portal — 沙箱应用弹窗

Flatpak/沙箱应用的文件对话框、截屏、通知等走这里。**原生应用不受影响。**

配置文件：`~/.config/xdg-desktop-portal/{,niri-,hyprland-}portals.conf`

```ini
[preferred]
default=gtk
org.freedesktop.impl.portal.FileChooser=gtk
org.freedesktop.impl.portal.ScreenCast=wlr
org.freedesktop.impl.portal.Screenshot=wlr
```

按 `XDG_CURRENT_DESKTOP` 自动选择桌面对应文件（`niri-portals.conf`、`hyprland-portals.conf`），无则读 `portals.conf`。

---

## 5. 环境变量 — CLI 工具回退

编辑器、浏览器、分页器：

```bash
# ~/.zshrc
export EDITOR=nvim
export VISUAL=nvim
export BROWSER=firefox
export PAGER=less
export TERMINAL=kitty
```

优先级低于 `xdg-open`，只影响直接读环境变量的 CLI 工具。

---

## 6. gsettings — GTK 传统默认终端（已弃用）

旧版 GNOME/Cinnamon 应用查这个来决定终端：

```bash
gsettings set org.cinnamon.desktop.default-applications.terminal exec 'kitty'
gsettings set org.gnome.desktop.default-applications.terminal exec 'kitty'
```

**不建议再用**。它已被 exo helpers（Thunar）或 desktop 文件规范（GNOME）替代。配了也不影响现代工具。

---

## 7. GTK_USE_PORTAL — 强制 GTK 走 portal

```bash
export GTK_USE_PORTAL=1
```

让原生 GTK 应用的文件对话框也走 xdg-desktop-portal（即用 portal 配置的 FileChooser 而非 GTK 原生对话框）。

**通常不应设置**，除非你在 Sandbox 环境或调试。原生应用走这个会导致异常（窗口层级问题、响应变慢）。

---

## 链路全景图

```
用户双击文件/文件夹
        │
        ▼
     xdg-open
        │
        ├── x-scheme-handler/http → xdg-mime → browser
        ├── inode/directory       → xdg-mime → file manager
        └── 其他 MIME 类型        → xdg-mime → 对应 app
                                            │
                                            ▼
  ┌─── 原生应用（GTK/Qt）直接打开，不走 portal
  │
  └─── Flatpak 应用：
            │
            ▼
    xdg-desktop-portal  → portals.conf → 对应后端 → 弹窗
                                            │
                                            ▼
  Thunar "Open Terminal Here"
        │
        ▼
    exo-open --launch TerminalEmulator → helpers.rc → kitty
```
