# Intel Xe-LP：先查设备，再分阶段配置

适用于阶段 01 分流、03 安装准备和 06 图形修复。**不能再凭“Intel 11 代以后”或 WhateverGreen 的旧支持范围，直接判定没有加速方案。** 部分 Xe-LP 已有 WhateverXe＋WhateverXe-Graphics 社区路线；这不等于所有 Iris Xe、UHD、Arc 或 Intel 新核显都支持。

1401 仍是默认 EFI 生成器。保留它生成的原版、审计和警告，再为实际设备补齐经核对的图形路线，不因此改为手写整份 EFI。原始 ACPI、安装网络和介质检查照常执行。

## 1 · 先选对路线

在 B 执行 `python scripts/solutions.py lookup gpu --target "8086:A7A0"`（替换为 A **物理设备** ID），或传 Windows 的 `PCI\VEN_8086&DEV_A7A0...`。脚本只提供查证入口，不能证明本机兼容。不要输入已经伪装后的 macOS device-id 当物理 ID。

按顺序核对：

1. 从 Windows 清单取 vendor/device/revision；核实 GPU 架构、EU 数、内存配置和显示布线。笔记本内屏 eDP／MIPI DSI、标准 DP、HDMI、USB-C、独显直连分别记录，不能从 CPU 型号猜输出路径。
2. 查 [WhateverXe 设备选择表](https://github.com/b00t0x/WhateverXe/blob/main/docs/device-selection.md)、[兼容性与实测](https://github.com/b00t0x/WhateverXe#compatibility)。本版路由表锁定于 [f26a368](https://github.com/b00t0x/WhateverXe/blob/f26a36880c49ea1063393d943527808c0f38feca/docs/device-selection.md)，包含 Tiger Lake、部分 Alder Lake／Twin Lake、Raptor Lake 的 Xe-LP ID，含反馈的 **8086:A7A0**。这是“已有后端”，不表示每个 ID／接口都通过实机测试。
3. 未命中本包离线表时，继续查当前原作者表，不立即判无解。仅有“Iris Xe”商品名时先补 ID。上述快照明确不支持 Rocket Lake；Meteor Lake 的 Xe-LPG、Xe2 及以后的架构不能套用。Arc 独显不因名字带 Xe 而进入此路线。
4. 核对完整 macOS 版本／build。此快照列出 Big Sur 11.7.11、Monterey 12.7.6、Ventura 13.7.8、Sonoma 14.8.9、Sequoia 15.8.x、Tahoe 26.7 及以后；不能简化成“Big Sur 到 Tahoe 任意版本”。未来版本仍重新核对原作者限制。
5. 把结论分成“作者后端覆盖／作者实测／社区同类实测／本机本轮实测”。同机旧备份优先，但版本、接口或内存变化仍需复核。没有可用安装显示输出时先解决安装阻塞，不能以“装完再修”跳过。

## 2 · 锁定一组组件，分清 EFI 与系统内安装

查 [WhateverXe](https://github.com/b00t0x/WhateverXe)、[WhateverXe-Graphics](https://github.com/b00t0x/WhateverXe-Graphics) 和依赖原作者的**同版本说明、发布资产与安装／卸载脚本**，记录提交、哈希、系统范围和回退方式。不要只下载一个 kext 就宣称完成。

| 位置 | 要核对什么 |
|---|---|
| EFI | Lilu、WhateverXe RELEASE、该组合要求的信任组件、启用顺序、内核条件、启动参数、SIP 配置 |
| macOS 系统内 | 配套 Graphics 的安装包、`/Library` 内容与 AuxKC、安装日志、实际加载与功能结果 |
| 备份和恢复 | 当前可启动 EFI、系统内组件卸载方式和所需文件、Windows／Recovery 的可用入口 |

2026-11-21 查到的 WhateverXe **0.8.3** 说明要求 Lilu ≥1.7.2、WEX 在 Lilu 之后、AMFIAnchor＋AMFIAnchorInjector，`csr-active-config` 为 Data `03080000`（0x803），启动参数 `-allow3d -disablegfxfirmware`。保留 AMFI；从旧 AMFIPass 1.4.1 Mod 迁移时按维护者说明移除旧项，不能同时混装。`amfi=0x80`、`ipc_control_port_options=0` 不作为通用要求。需要因其他根补丁保留 OCLP AMFIPass 时按 [AMFIAnchor 的顺序与条件](https://github.com/b00t0x/AMFIAnchor) 查证。

**当前有文档版本差异：** Graphics `v0.4.4` 标签 README 仍写旧 Mod，主分支已经改为 AMFIAnchor；反馈案例则是 WEX 0.8.2＋Graphics 0.4.3。Agent 必须查变更记录、实际发布包及维护者说明，确认要采用的成套组合。不能把三个时间点拼成一套“已验证安装命令”。有矛盾时先记为待查，继续其他设备，不盲目扩大 SIP／AMFI 关闭范围。

采用当前路线时：

- **WhateverXe 与 WhateverGreen 不得同时生效。** 不看文件夹里有没有，而是看 `Kernel → Add` 的 Enabled、Arch、MinKernel、MaxKernel 和实际加载。禁用旧项并保留备份，再核对依赖和同版本 ocvalidate。多 GPU 机器也要审查整套补丁的冲突，不能为每张卡各装一个后全开。
- 通常让 WEX 按物理 ID 自动选择，按作者说明移除过时的 ICL／其他 GPU 伪装；不要仅为改显卡显示名称伪装 ID，也不能靠 spoof 把不支持架构变为支持。
- DVMT、aperture、接线按本机和作者说明选择。作者建议可设置时 DVMT ≥96MB，但有固定 60MB 的专门路线；不要把案例的 128MB／256MB 写成所有主板必须值，不擅自解锁或修改未知 BIOS 变量。
- 系统内组件按该版本安装方法处理，通常进入桌面后安装 Graphics、检查 `/var/log/install.log`、完成必要授权再重启。当前说明是 `/Library`＋AuxKC，不改 System 快照；若曾使用旧 Xe 系统补丁，先按作者方式恢复，不能叠加安装。根卷仍密封也不等于 SIP 完全启用。
- 原作者 AutoPkg 路线涉及完整安装介质的 `Library/Packages`、额外组件和不同 SIP 条件。**不默认用于本流程的 BaseSystem 联网恢复 U 盘**；只有确认介质类型与原作者前提都满足才单独采用。不能把 AutoPkg 包当普通安装包手动运行，也不能混用其他项目的同名资源。

`efi_audit.py` 会列出可识别名称的 WEX／WEG 启用区间重叠；阶段 03 安装前检查与 06 清单封口会报 `GPU_PATCH_CONFLICT`。重命名 bundle、其他补丁冲突、二进制与运行期加载仍需 Agent 核对，静态检测不是全面兼容性保证。有效区间不重叠或一项已禁用不会单凭文件存在报冲突；不明条件须先修正再复核。

## 3 · 安装前给用户一句明白话

按 [安装能力告知](13-install-expectations.md) 登记这张卡到 `postinstall_items`，例如：

> **现在**　已准备这台电脑的 Xe 核显安装配置。安装界面可能较慢，完整加速需要进入桌面后再补装图形组件。
>
> **接下来**　装好系统后我会继续修复并验证显卡，再检查其他硬件。
>
> **请你**　照安装清单操作；如果黑屏、卡住或无法操作，拍照反馈。

仅在安装显示、键鼠、存储、联网条件确已核对时使用这段话。“预计没有加速”不等于黑屏／内核崩溃正常，也不代表以后一定可修好。阶段 06 保留同一设备 ID，不另起任务遗失它。

## 4 · 本轮验收后回到全机清单

先按 [驱动修复与故障恢复](06-postinstall.md) 保存基线、发恢复短卡。重启失败先区分显示黑屏与 SSH 失联，再回 Windows／Recovery 取证或回退；恢复 EFI 不能代替卸载系统内 Graphics。

每项单独记录：

1. **加载与显示：** 对应驱动、物理 GPU、Metal 枚举、实际输出接口／分辨率／刷新率／HiDPI。显示“1536MB”或“Metal 2”只证明相关枚举，不能代替运行测试。
2. **计算：** 在 A 执行现有 Metal 探针并核对目标 GPU、命令完成和结果回读；脚本选到另一张 GPU 时重测。通过后只登记该项 Metal 计算通过。
3. **实际使用：** 窗口绘制、缩放、视频解码、需要时编码、每个要用的输出和唤醒，分别测试。Graphics 0.4.4 的说明修复了 0.4.3 的 H.264 编码回归，不能把旧案例的 Metal 通过扩写为编码已通过。
4. **限制：** 当前作者仍列有 DP 热插拔／USB-C、部分内屏、睡眠等限制或未测范围，FairPlay 也不支持；按使用接口实际确认。不强行热插拔正在使用的唯一输出，先给出恢复办法。一个 DP 4K60 案例不代表 HDMI、多屏或睡眠都通过。
5. 更新 GPU 行后，立即回 06 队列继续各网口、无线／蓝牙、USB、音频与系统功能。暂不能修的项保留原因和待办；全部条目收尾、限制明确接受后，才进入 07 硬盘引导。

## 反馈案例与可迁移的经验

[反馈 #1](https://github.com/zxmls-ai/Hackintosh-skill/issues/1) 报告 i7-1370P／Iris Xe `8086:A7A0`、Tahoe 26.7.1 (25G241)、标准 DP 下 4K60 HiDPI 和 Metal 枚举可用。其 [公开配置快照](https://github.com/nova286/RaptorLake-P-i7-1370P-MiniPC-Hackintosh-EFI/tree/a9480da7c24911015307b8a29f87943ccfd7ba44) 是社区实测线索；本 Skill 维护侧没有复测这台机器或取得本轮 Metal 计算回读。不得直接复制身份、ACPI、USB 映射或把“全模块”扩写为所有功能已验收。

同一反馈中的其他硬件也继续逐项查证：

- Intel 蓝牙：核对 USB ID、内部端口映射、固件、BlueToolFixup、内核启用范围，再做扫描／配对／睡醒测试。案例提到 2.5.0，但本次读取的 OpenIntelWireless 官方 latest 仍是 2.4.0；需要查清具体 fork／开发提交和资产。改 `MaxKernel` 仅改变启用范围，不证明新 macOS 支持，也不保证接力。
- 音频：Conexant codec 的 VoodooHDA 是候选。案例是 3.6.8 的系统内路线；当前原作者说明已经出现 3.7.0x EFI 注入路线，不能强制沿用旧 AuxKC／安全设置。按所选版本排除同设备 AppleALC 等冲突，分别试听与录音回放。
- 两张有线网卡分两行，分别核对 PCI 匹配、实际网络通信。任一 GPU／无线项目受阻都不停止整机清单。

## 本版核对基准

- [WhateverXe f26a368（含 0.8.3 说明）](https://github.com/b00t0x/WhateverXe/tree/f26a36880c49ea1063393d943527808c0f38feca)
- [Graphics cf0a218 主分支快照](https://github.com/b00t0x/WhateverXe-Graphics/tree/cf0a21841d0464dd0b0e9c5b1db28d47137a6832) 与 [v0.4.4 标签说明](https://github.com/b00t0x/WhateverXe-Graphics/blob/v0.4.4/README.md)
- [IntelBluetoothFirmware 发布](https://github.com/OpenIntelWireless/IntelBluetoothFirmware/releases)、[安装说明](https://openintelwireless.github.io/IntelBluetoothFirmware/Installation.html)
- [VoodooHDA 原作者说明](https://github.com/CloverHackyColor/VoodooHDA/blob/main/readme.md)

查阅日期 2026-11-21。执行时仍检查变化，选定后锁版本。本包不捆绑第三方图形驱动，不宣称其适用所有 Xe 设备。
