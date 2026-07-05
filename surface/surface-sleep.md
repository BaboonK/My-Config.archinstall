## Surface Pro 8 休眠问题

### 1. 检查和调试

```bash
# 查看支持的睡眠模式（Surface 一般只有 s2idle，无 S3 deep）
cat /sys/power/mem_sleep               # 期望 [s2idle]
cat /sys/power/state                   # 期望 包含 freeze mem disk

# 查看哪些设备允许唤醒系统
cat /proc/acpi/wakeup
# ACPI 唤醒源
grep '\*enabled' /proc/acpi/wakeup
```

### 2. 电源键与合盖配置（已执行）

`/etc/systemd/logind.conf`：

```ini
[Login]
HandlePowerKey=suspend                 # 单击电源键 → 挂起
HandleLidSwitch=suspend                # 合盖 → 挂起
HandleLidSwitchExternalPower=suspend   # 接电源时合盖也挂起
HoldoffTimeoutSec=1s                   # 恢复后合盖重新触发的冷却时间（默认 30s）
InhibitDelayMaxSec=1s                  # NetworkManager 等 inhibitor 最大延迟（默认 5s）
```

> 建议重启一次系统

### 3. surface-control 电源优化（d3cold 子命令已废）

> **重要订正**：`surface-control` v0.5.0 已移除 `d3cold` 子命令，下列旧命令会报错。

```bash
# 新版（surface-control 0.5+）——可选电源管理
# 列出可用 profile
sudo surface profile list
#   low-power / balanced / balanced-performance / performance

# 切换（如电池下切 low-power，与 power-profiles-daemon 联动）
sudo surface profile get
sudo surface profile set low-power
# 或者用 power-profiles-daemon:
powerprofilesctl get                          # balanced
powerprofilesctl set power-saver
```

注：surface-control 包名是 `surface-control`，但二进制可执行文件在 `/usr/bin/surface`

### 4. 已知限制

- Surface 仅支持 s2idle（S0ix Modern Standby），不支持 S3 deep sleep
- 键盘（Surface Keyboard，非 USB 连接）在 s2idle 下不支持唤醒，需开盖或按电源键唤醒
- 部分 iptsd 和触控驱动恢复时可能出现短暂的 IO 错误日志——iptsd 会自动恢复，无需 resume hook



### 真正 hibernate（休眠到磁盘）— 待规划

当前默认不配置，仅记录思路。

#### 为什么 zram 不能用于 hibernate

| 类型 | 存储位置 | 断电后 | 用途 |
|---|---|---|---|
| zram swap | RAM（压缩） | 数据丢失 | 扩展可用内存 |
| swapfile | 磁盘 | 数据保留 | hibernate |

**Hibernate 的目的**是将 RAM 内容写入**持久存储**，以便断电后恢复。zram 存储在 RAM 中，断电后数据丢失，无法用于 hibernate。

当前系统只有 zram swap（7.7G），需要在磁盘上创建 swapfile 才能启用 hibernate。

#### 实现步骤

```bash
# 1. 创建 swapfile 目录
sudo mkdir -p /swap

# 2. btrfs 创建 swapfile（≥ 内存大小 16G）
sudo btrfs filesystem mkswapfile --size 16G /swap/swapfile

# 3. 启用 swapfile
sudo swapon /swap/swapfile

# 4. 找 swapfile 的物理偏移（用于 resume_offset）
sudo btrfs inspect-internal map-swapfile /swap/swapfile

# 5. /etc/mkinitcpio.conf 添加 resume hook
#    HOOKS=(... block resume filesystems fsck)
sudo mkinitcpio -P

# 6. /etc/default/grub 补 resume / resume_offset
#    GRUB_CMDLINE_LINUX_DEFAULT="... resume=UUID=<根分区 UUID> resume_offset=<上一步 offset>"
#    根分区 UUID: 0e29e6c4-3712-4642-aeae-8b892e59e4a1
sudo grub-mkconfig -o /boot/grub/grub.cfg

# 7. /etc/fstab 添加 swapfile 条目（可选，开机自动挂载）
#    /swap/swapfile none swap defaults 0 0

# 8. /etc/systemd/logind.conf 启用 suspend-then-hibernate
#    HandleLidSwitchExternalPower=suspend-then-hibernate
sudo systemctl restart systemd-logind
```

#### 注意事项

- btrfs + swapfile + hibernate 内核 6.x 已支持
- swapfile 必须在单一 device、无压缩、无快照干扰（`mkswapfile` 自动处理）
- 仍只有 s2idle（Modern Standby），hibernate 是 disk state（S4 到 swapfile），与 S3 deep 不同
- 需要 33G 可用空间（当前有 32.4G，可能需要清理快照）
