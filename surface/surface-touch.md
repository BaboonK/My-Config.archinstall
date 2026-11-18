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

GNOME 风格的触摸圆圈指示器，触摸时在触摸点显示白色半透明圆圈，松手后淡出消失。

| 文件 | 路径 | 用途 |
|---|---|---|
| 主程序 | `/usr/local/bin/touch-indicator.py` | 读取 evdev + GTK3 layer-shell 画圆圈（`root:root 755`） |
| udev 规则 | `/etc/udev/rules.d/99-touch-indicator.rules` | 允许 input 组读取 IPTSD 触摸设备 |
| systemd 服务 | `~/.config/systemd/user/touch-indicator.service` | 用户级服务，由 Hyprland 启动 |
| 手动 launcher | `~/.local/bin/touch-indicator` | 无 sudo wrapper，要求当前用户在 input 组 |

脚本按设备名 `IPTSD Virtual Touchscreen` 自动查找 `/dev/input/eventN`。不要硬编码 event 编号；重启、睡眠恢复或设备重枚举后 event 编号可能变化。

当前启动方式：

```lua
-- ~/.config/hypr/hyprland.lua
hl.on("hyprland.start", function()
    hl.exec_cmd("systemctl --user restart touch-indicator.service")
end)
```

```kdl
// ~/.config/niri/niri.lua
spawn-sh-at-startup "systemctl --user restart touch-indicator.service"
```

`touch-indicator.service` 本身保持 `disabled`，避免 GNOME session 或 Wayland 环境未就绪时自动启动；进入 Hyprland 后由上面的 autostart 拉起。

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

> 需要重新登录才能让 input 组生效（udev 规则要求 input 组权限读取触摸设备）。主程序 root 拥有，但运行进程是普通用户，不需要 sudo。


