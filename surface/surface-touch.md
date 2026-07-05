## Surface Pro 8 触摸屏与手写笔

### 1. 触控灵敏度（iptsd）

底层灵敏度由 iptsd 的 `/etc/iptsd.conf` 控制。修改后需重启服务。

```bash
sudo systemctl restart iptsd@dev-hidraw7.service
```

| 参数 | 默认值 | 说明 |
|---|---|---|
| `ActivationThreshold` | 24 | 触碰多轻就被识别（越低越灵敏） |
| `DeactivationThreshold` | 20 | 手指抬起多快被识别（越低越灵敏） |
| `PositionThresholdMin` | 0.04cm | 多小的移动就被识别为"移动" |
| `SizeThresholdMin` | 0.1cm | 多小的面积变化就被识别 |

> 当前值 = Windows 默认灵敏度（24/20），由 iptsd 逆向工程得出。

### 2. 触摸指示器（touch-indicator.py）

GNOM 风格的触摸圆圈指示器，触摸时在触摸点显示白色半透明圆圈，松手后淡出消失。

| 文件 | 路径 | 用途 |
|---|---|---|
| 主程序 | `/usr/local/bin/touch-indicator.py` | 读取 evdev + GTK3 layer-shell 画圆圈（root 拥有） |
| udev 规则 | `/etc/udev/rules.d/99-touch-indicator.rules` | 允许 input 组读取 IPTSD 触摸设备 |
| systemd 服务 | `~/.config/systemd/user/touch-indicator.service` | 用户级服务，自动启动 |

管理命令：

```bash
systemctl --user start touch-indicator    # 启动
systemctl --user stop touch-indicator     # 停止
systemctl --user restart touch-indicator  # 重启
systemctl --user status touch-indicator   # 查看状态
```

配置调整（编辑 `/usr/local/bin/touch-indicator.py` 顶部常量）：

```python
CIRCLE_RADIUS = 18        # 圆圈半径（像素）
CIRCLE_COLOR = (1,1,1,0.5)  # 圆圈颜色（RGBA，白色 50% 透明）
BORDER_COLOR = (1,1,1,0.8)  # 边框颜色（RGBA）
BORDER_WIDTH = 2           # 边框宽度
FADE_DURATION_MS = 300     # 淡出时间（毫秒）
```

> 需要重新登录才能让 input 组生效（udev 规则要求 input 组权限读取触摸设备）。

### 3. 触屏手势（Hyprgrass 插件）

Hyprland 0.55 内建的触摸屏能力只有**边缘滑动切工作区**（`gestures:workspace_swipe_touch`），缺 pinch、longpress、tap、非边缘 swipe 等手势。hyprgrass 是 Hyprland 官方插件生态中专为触摸屏（不是触控板）写的插件。

#### 手势类型

| 手势 | 语法 | 用途 |
|---|---|---|
| `swipe` | `{kind="swipe", fingers=3, direction="down"}` | 多指滑动 |
| `edge` | `{kind="edge", origin="right", direction="left"}` | 从屏幕边缘内滑 |
| `longpress` | `{kind="longpress", fingers=2}` | 长按 |
| `tap` | `{kind="tap", fingers=3}` | 轻击 |
| `pinch` | `{kind="pinch", fingers=3, direction="pinchin"}` | 捏合 |

#### 当前绑定的手势

```lua
-- 三指下滑关闭窗口
hl.plugin.hyprgrass.gesture {
    pattern = {kind = "swipe", fingers = 3, direction = "down"},
    action = "close",
}

-- 三指上滑全屏
hl.plugin.hyprgrass.gesture {
    pattern = {kind = "swipe", fingers = 3, direction = "up"},
    action = "fullscreen",
}

-- 二指长按拖动窗口
hl.plugin.hyprgrass.bind {
    pattern = {kind = "longpress", fingers = 2},
    action = hl.dsp.window.drag(),
    mouse = true,
}
```

#### 安装

```bash
# 构建依赖
sudo pacman -S --needed glm meson ninja

# 第一次跑先 update headers（hyprpm 检测 hyprland 头文件版本）
hyprpm update

# 仓库 + 启用插件
hyprpm add    https://github.com/horriblename/hyprgrass
hyprpm enable hyprgrass

# 验证
hyprpm list                                  # 期望：hyprgrass enabled=true
hyprctl plugin list                          # 期望：列出 hyprgrass by horriblename

# 首次启用时键盘在手，万一触摸失灵直接禁用
hyprpm disable hyprgrass
```

> `hyprgrass-git` AUR 包依赖 `hyprland-git`，会替换你整个 Hyprland，不推荐。用 `hyprpm` 路线绕开 AUR 包，用当前官方 hyprland 的 headers 自构建。

#### 配置（`~/.config/hypr/hyprland.lua`）

```lua
-- 插件参数
hl.config({
    plugin = {
        hyprgrass = {
            sensitivity                 = 4.0,    -- 触屏建议 4.0
            long_press_delay           = 400,    -- 长按触发时间（ms）
            resize_on_border_long_press = false,
            edge_margin                 = 40,     -- 边缘触发区域（像素）
        },
    },
})
```

启动时自动加载（在 `hl.on("hyprland.start", ...)` 内）：

```lua
hl.exec_cmd("hyprpm reload -n")
```

完整手势语法见 [hyprgrass 文档](https://github.com/horriblename/hyprgrass/blob/main/docs/configuration.md)。

### 4. Hyprland 配置参考

#### 触摸屏与笔设备绑定

```lua
hl.config({
    input = {
        touchdevice = {
            enabled  = true,
            transform = 0,
            output   = "eDP-1",
        },
        tablet = {
            transform = 0,
            output   = "eDP-1",
        },
    },
})
```

#### 边缘滑动切工作区（Hyprland 内建）

```lua
hl.config({
    gestures = {
        workspace_swipe_touch       = true,   -- 触摸屏边缘滑动主开关
        workspace_swipe_invert      = true,   -- 手指右滑→右边工作区
        workspace_swipe_cancel_ratio = 0.2,   -- 滑不到 20% 就回弹
    },
})
```

> `workspace_swipe_touch` 是**边缘滑动**（从屏幕最左/右边缘开始），不是三指滑动。三指滑动是触控板手势（`hl.gesture`）。

### 5. 回退与故障排除

```bash
# 禁用 hyprgrass
hyprpm disable hyprgrass && hyprpm remove hyprgrass
# 删 hyprland.lua 中 plugin 段、bind、autostart 内 hyprpm reload 一行
hyprctl reload

# Hyprland 升级后 ABI 失配则重新构建
hyprpm update

# 重置 iptsd 灵敏度为默认
sudo sed -i \
    -e 's|^ActivationThreshold = .*|# ActivationThreshold = 24|' \
    -e 's|^DeactivationThreshold = .*|# DeactivationThreshold = 20|' \
    /etc/iptsd.conf
sudo systemctl restart iptsd@dev-hidraw7.service

# 停止触摸指示器
systemctl --user stop touch-indicator
systemctl --user disable touch-indicator
```
