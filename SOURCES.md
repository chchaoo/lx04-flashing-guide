# 文件来源与许可

预打包的 Release 已移除。以下是固件、工具与应用的上游来源，供手动准备文件或核对已有本地包使用。

| 内容 | 版本 / 来源 |
|---|---|
| mtkclient（完整未修改源码） | [bkerler/mtkclient](https://github.com/bkerler/mtkclient/tree/cd25cf9c1ff6d36e82697ac2c798e69e9cfb78c3)，commit `cd25cf9c1ff6d36e82697ac2c798e69e9cfb78c3`，GPL-3.0 |
| 分区表、preloader、Magisk boot | 本地原始 `xiaomi-x04g-lx04-main.zip`，来自 [AyraHikari/xiaomi-x04g-lx04](https://github.com/AyraHikari/xiaomi-x04g-lx04) |
| X04G 固件 | 本地原始 `dump_mico_x04g.tar.xz`，来自 [Internet Archive](https://archive.org/details/dump_mico_x04g)；28 个分区文件已逐一核对原包 SHA-256 |
| Python | [Python 3.12.10 Windows embeddable x64](https://www.python.org/downloads/release/python-31210/)，保留原包许可 |
| ADB | [Android Platform-Tools](https://developer.android.com/tools/releases/platform-tools)，37.0.1-15733141，保留 NOTICE.txt |
| UsbDk | [daynix/UsbDk](https://github.com/daynix/UsbDk)，1.0.22 x64，本地原安装包 |
| MTK / Google USB 驱动 | 本地原 FlashTool 驱动及 Google USB Driver r13，保留原文件 |
| KISS Launcher | F-Droid `fr.neamar.kiss`，3.26.0 (224)，最低 Android 5.0 |
| Simple Keyboard | F-Droid `rkr.simplekeyboard.inputmethod`，6.8 (149)，最低 Android 7.0 |
| VirtualSoftKeys | F-Droid `tw.com.daxia.virtualsoftkeys`，0.2.0.180212.1 (10)，最低 Android 4.4 |

教程及本仓库自编脚本使用 MIT 许可。固件、驱动、运行库和应用保持各自许可，不适用本仓库的 MIT 许可。包内没有个人设备备份、日志或本地设备状态文件。

使用本地完整包时，mtkclient 源码位于 lx04/mtkclient/，运行库的许可和版权声明位于 runtime/Lib/site-packages/。从上游重新准备文件时，请一并保留相应源码、许可证与来源记录。

命令行运行库版本（未包含 Qt 图形界面）：

- capstone 5.0.9
- colorama 0.4.6
- keystone-engine 0.9.2
- mfusepy 3.1.1
- pycryptodome 4.3.0
- pycryptodomex 3.24.1
- pyserial 3.5
- pyusb 1.3.1
- unicorn 2.1.4
