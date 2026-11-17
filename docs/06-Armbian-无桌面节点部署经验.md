# Armbian 无桌面管理节点：E900V21C / S905L / P211 实机经验

> 这是一台山东联通版创维 E900V21C 的实测记录，不是所有 E900V21C 或 S905L 盒子的通用刷机教程。本文只发布可复用经验，不分发固件、Bootloader、FIP、分区镜像或私人配置。

## 1. 实机身份与已验证系统

| 项目 | 实测 |
|---|---|
| 设备 | 创维 E900V21C（山东联通版） |
| SoC / 板型 | Amlogic S905L / GXL，P211，Venz V10 |
| 内存 / 存储 | 标称 1GB RAM / 8GB eMMC |
| 系统 | ophub Armbian 26.11.0 / Debian 13 Trixie |
| 内核 | Linux 6.18.59-ophub |
| 架构 | ARM64，Cortex-A53 四核 |
| DTB | `meson-gxl-s905l-venz-v10.dtb`（ophub 设备数据库 ID 116） |
| 启动 | 已验证拔除安装 U 盘后的 eMMC 独立冷启动 |

**不要只按 SoC 名称选镜像。** 同为 S905L 的盒子可能使用不同 PCB、设备树、FIP、分区布局和无线芯片。P211 不应套用 P212、S905L3A 或 M201S 的 DTB/Bootloader。

## 2. 启动链与 eMMC：本机踩坑

- 优先沿用 ophub 官方设备支持、镜像和 `armbian-install`，避免自行构造整套启动流程或增加桌面。
- 原机专属 FIP 是板级启动关键材料。写 eMMC 前，至少备份原始 FIP、完整 U-Boot 环境和 eMMC 起始区域；绝不把其他盒子的通用 FIP 写进来。
- U-Boot 环境修改前先备份完整环境；写后检查 CRC、读回内容和哈希。不要把 `saveenv`、整区擦除或一次性刷写脚本当成无害操作。
- 本机第一次 eMMC 安装后曾出现 FAT 文件头归零、ext4 元数据校验错误。后续在重建安装阶段关闭 eMMC 内部缓存后，文件系统检查通过并成功冷启动；冷启动后硬件缓存状态自动恢复为启用。**这是单台机器的经验，不是所有 DG4008/eMMC 的通用修复命令**，不能未经诊断就永久关闭缓存。
- 真正的安装验收必须拔掉安装 U 盘并完全断电冷启动。确认根分区和 /boot 均来自 eMMC，再检查文件系统、FIP/U-Boot 读回、网络和 systemd。安装器返回成功或从 USB 启动成功都不能代替这一步。

## 3. 网络和无线能力边界

| 子系统 | 实测边界 |
|---|---|
| Ethernet | st_gmac，10/100BASE-T；单向 TCP 约 94Mbps，不是千兆 |
| Wi-Fi | RTL8189FTV/FS，SDIO，2.4GHz 802.11b/g/n，1T1R；不支持 5GHz / Wi-Fi 5 / Wi-Fi 6 |
| Wi-Fi 吞吐 | 本机未关联 AP，因此没有可引用的实际 TCP/UDP 吞吐数据；PHY 标称速率不等于应用吞吐 |
| USB | 仅枚举 USB 2.0 High-Speed，未验证 USB 3.0 |
| Bluetooth | 未枚举 HCI；RTL8189FTV 是 Wi-Fi-only。未拆板，不能绝对排除另有独立蓝牙芯片 |
| Audio | 系统可枚举声卡/模拟 Codec，但 3.5mm AV 模拟音频未完成实物验收 |

### RTL8189FS 驱动

本机在匹配的 Linux 6.18.59-ophub 内核头文件和 ARM64 工具链下构建并加载了 `8189fs` 模块。源码参考 `jwrdegoede/rtl8189ES_linux`，构建时需要针对新内核 cfg80211 接口变化做兼容处理。

- 模块版本：`v5.7.9_35795.20191128`
- 实机模块 SHA256：`afc3720c36ac5284c8a7c21a60dbbab0e7c9c193f3cb669760746fa49ce89827`
- 曾扫描到周边 AP，但扫描路径存在不稳定现象；没有关联 AP，也没有宣称 Wi-Fi 性能已验收。
- 内核升级后必须按新内核头文件重新构建/验证模块，不能把旧 `.ko` 直接复制到新内核。

## 4. 轻量节点定位

- 实测可用 RAM 约 784MiB；适合 SSH/ADB、轻量控制代理和单一边缘服务。
- 不适合千兆软路由、NAS、大型容器、重编码/转码或高并发双向转发。
- ADB server 只绑定设备本机 `127.0.0.1:5037`，由可信 SSH 管理链路调用；不要绑定 `0.0.0.0` 或暴露到公网。
- SSH 使用密钥认证。保留 NetworkManager、chrony、硬件 watchdog 等必要基础服务，不为追求“极简”而随意裁剪。
- 温度传感器曾报告明显不合理的 0–1°C，不能据此判断真实温度或散热余量。

## 5. 可复用验收清单

1. 核对 PCB/SoC/设备树，确认确切板型；不要按外壳或“S905L”泛称选镜像。
2. 备份并校验原机 FIP、U-Boot 环境和 eMMC 起始区域。
3. 使用对应设备条目的官方镜像和 DTB，保持最小改动。
4. 安装后拔除 U 盘、断电冷启动，确认 root 与 /boot 都来自 eMMC。
5. 检查文件系统、FIP/U-Boot 读回、网口、systemd 和 ADB 监听地址。
6. 对未实际测试的 Wi-Fi 吞吐、蓝牙、模拟音频、4K/转码、AirPlay2 和长期稳定性明确标注“未验证”。

## 6. 隐私与分发边界

本仓库不发布本机 FIP、Bootloader、eMMC/Android 分区、定制 Armbian 镜像、ADB 密钥、MAC/IP、IPTV 源、Token 或私人网络配置。复现者应使用自己有权使用的上游镜像和自己的原机备份。

## 参考

- [ophub/amlogic-s9xxx-armbian](https://github.com/ophub/amlogic-s9xxx-armbian)
- [Realtek RTL8189FTV](https://www.realtek.com/Product/Index?cate_id=194&id=600)
- [Android 6.0.1 实机适配与救砖笔记](../README.md)
