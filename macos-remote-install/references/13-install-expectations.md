# 安装前说明：哪些功能装完系统再修

**EFI 构建和写盘检查完成后、让用户重启安装前，必须给一次简短说明。** 对照 A 的硬件报告、实际生成配置、工具日志和原作者安装方式，区分“安装必需条件”“本轮安装后再修”“支持情况待查”。生成器自带审计与 ocvalidate 证明的是配置检查结果，不等于驱动已经在 macOS 加载或所有功能已实测。

## 判断顺序

1. 先核对安装阶段能否显示画面、操作键鼠、访问目标存储，以及联网恢复所需的网络路径。任一必需条件缺失，属于安装阻塞，先修复或给出经过核对的替代路径；不能把唯一网卡未驱动也笼统推到装完再修。
2. 对每项未完整配置的硬件，记录具体 ID、当前配置、维护者来源、为何留到系统内安装、预期现象及下一步。没有查明支持情况时写待查，不承诺“装完肯定能修好”。如果已是经过验证的安装后修复方案，暂不需要为该项反复重做安装 EFI。
3. 展示下面的短卡并保存交接记录；已知待修条目进入 `postinstall_items`。阶段 06 接通 macOS 后，先把它们并入全机设备清单，再扫描其他硬件，逐项确认。延期修复不是功能通过，也不是用户已接受永久限制。

## 给用户的短卡

以下是**本轮确实识别到对应情况时**的示例，不能每台机器照搬：

> **03 · 启动 U 盘已准备好**
>
> **现在**　EFI 和安装介质检查已通过。下面两项会在进入 macOS 桌面后继续处理：
>
> | 项目 | 安装时可能看到什么 | 后续安排 |
> |---|---|---|
> | NVIDIA 显卡加速 | 可显示画面，但动画和缩放可能较慢 | 在 macOS 内安装适配驱动并验证加速 |
> | RTL8127 网口 | 本轮尚不能用这个网口 | 安装先用已核对的备用网口，进入桌面后继续修复 |
>
> **接下来**　先安装 macOS，之后自动继续检查和修复整机硬件。这些已说明的功能缺口，不能单独作为 EFI 制作失败的依据。
>
> **请你**　按接下来的启动步骤操作，保留 U 盘；出现未说明的卡死或报错时拍照反馈。

如果本轮没有已知延后项目，只说明“未发现需要明确延后的项目，进入桌面后仍会逐项实测”，不要罗列一堆假想问题。网络仍受阻时改成“安装前还需解决联网”，不能使用“已准备好”版本。

**NVIDIA 示例的依据：** [1401 README](https://github.com/nullmoth/1401#using-it) 与 [NullMoth 驱动](https://github.com/nullmoth/nvidia-macos-driver) 把现代 NVIDIA 驱动安装放在 macOS 启动后；EFI 中有启动参数、安装 U 盘中有驱动附件，并不表示系统内已安装驱动。必须按实际 GPU、系统 build、版本和显示布线查证。

**RTL8127 示例的边界：** 这是用户本轮生成产物的缺项，不写成“所有 1401 版本都不含 RTL8127”或“此卡只能在装完后驱动”。按当前 kext、设备匹配、x86-64 架构与具体发布资产查 [RTL812xLucy](https://github.com/stefb69/RTL812xLucy) 等维护者项目。若可在安装 EFI 中正确配置就不必延后；只有另有已核对的安装联网路径时，才把未完成的此网口转入 06。

## Agent 的固定记录

**Xe-LP 也按同一份清单交接。** 对已核对适用 WhateverXe＋Graphics 的机器，说明 EFI 插件与 macOS 内图形组件的区别，登记安装时可能无完整加速和 06 的补装／验证任务。采用 [Xe 专用分支](14-xe-lp.md)，不把其完整介质 AutoPkg 方法套在联网恢复 U 盘。唯一显示输出不可用时先解决安装阻塞，不能延后。

阶段 03 review 的 `checks.installation_expectations` 必须为 `pass`，`manifest` 指向相对私有 run 的 `install-expectations-v1` 文件。校验引用文件及本机身份，把说明绑定到本次实际安装配置。控制器验证结构和字节；说明是否真实、安装条件是否成立仍由 Agent 依据原始证据核对。

```json
{
  "schema": "install-expectations-v1", "machine_uuid": "本任务 A UUID",
  "config": "staging/install/EFI/OC/config.plist",
  "assessment": "evidence/install-capability-review.txt",
  "presented": "evidence/install-expectations-presented.txt",
  "install_network": "evidence/installer-network-path.txt",
  "install_display": "evidence/installer-display-and-input.txt",
  "install_input": "evidence/installer-display-and-input.txt",
  "install_storage": "evidence/installer-storage-path.txt",
  "blocking_items": [],
  "postinstall_items": [
    {
      "id": "gpu-primary", "category": "gpu", "name": "本机具体显卡",
      "hardware_id": "本机稳定硬件标识",
      "reason": "原作者要求系统启动后安装驱动组件",
      "impact": "预计安装时可显示但没有完整加速，动画可能较慢",
      "defer_to": "06-devices", "next_action": "按已锁定驱动安装并做功能测试",
      "source": "https://github.com/nullmoth/nvidia-macos-driver",
      "evidence": "evidence/gpu-install-route.txt"
    }
  ]
}
```

路径必须有实际内容，`presented` 是已展示说明的记录，不是尚未发出的草稿。`config` 必须与 ACPI 验收指向的安装配置字节一致；改配置后重新评估并保存新文件。`blocking_items` 非空不能通过；这时保留阻塞清单并解决问题，而不是删除行强行过关。`postinstall_items` 可以为空，但 `assessment` 仍须说明本轮核对依据。

已延后条目的 `id/category/hardware_id` 在阶段 06 清单中保留。控制器拒绝漏掉或换名隐藏这些设备；其状态初始按本轮实际情况填写 `pending` 等，只有后来真正功能测试通过才记 `passed`。这份清单仅是起点，不能替代阶段 06 的全机扫描、另一张网卡／无线／USB／睡眠等检查。

旧任务已在 04 或更后阶段时，不为补通知退回重装；在当前交接中补上实际待修项目，进入或继续 06 全机清单。新执行的 03 review 才适用新门槛。
