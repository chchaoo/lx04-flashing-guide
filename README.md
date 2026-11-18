# 小米小爱触屏音箱 LX04 刷机

把国行 **LX04** 刷成 **X04G（Android 10）**，再装好桌面、键盘和返回键。仅适用于机身标签为 LX04 的设备。

需要 Windows 10/11 **64 位**电脑、Micro-USB **数据线**、5V / 2A 电源和至少 15 GB 空间。刷机会清空数据，第 3 步备份完成并核对后再写入。刷机文件和备份都放在 `D:\lx04` 下，先确认 D 盘可用空间。

## 1. 下载原始文件并准备目录

打开 [v1.0.0 Release](https://github.com/chchaoo/lx04-flashing-guide/releases/tag/v1.0.0)，下载下面的附件。先在 D 盘根目录创建 `D:\lx04`，再按表格放置各个附件。

| 附件 | 放置方法 |
|---|---|
| `python-3.12.10-amd64.exe` | 安装 Python 3.12，保留 Python Launcher（`py`） |
| `mtkclient-cd25cf9.zip` | 解压，将源码目录重命名为 `D:\lx04\mtkclient`，其中应有 `mtk.py` |
| `xiaomi-x04g-lx04-main.zip` | 将其 `files` 目录放到 `D:\lx04\files` |
| `dump_mico_x04g.tar.xz` | 解压到 `D:\lx04\global`，这一层直接包含 `boot.bin` 等分区文件 |
| `platform-tools_r37.0.1-windows.zip` | 解压得到 `D:\lx04\platform-tools\adb.exe` |
| `UsbDk_1.0.22_x64.msi` | 放到 `D:\lx04\drivers` |
| `usb_driver_r13-windows.zip` | 将其中 `usb_driver` 目录重命名为 `D:\lx04\drivers\Google` |
| `KISS.apk`、`SimpleKeyboard.apk`、`VirtualSoftKeys.apk` | 放到 `D:\lx04\apps` |

另外解开 `D:\lx04\files\FlashTool.7z`，将其中 `FlashTool\driver` 目录的内容放到 `D:\lx04\drivers\MTK`，应能看到 `Mediatek-Drivers-Install.bat`。使用能够解开 ZIP、7z 和 tar.xz 的解压工具。不要把固件再多套一层目录。

打开 **命令提示符（cmd）**，准备目录并安装命令行依赖：

```cmd
cd /d D:\lx04
mkdir backup\partitions
py -3.12 -m pip install pyusb pycryptodome pycryptodomex colorama pyserial mfusepy keystone-engine capstone unicorn
py -3.12 mtkclient\mtk.py --help
```

能显示帮助后继续。后文都在 `D:\lx04` 下执行，路径不要有中文或空格。附件不包含任何个人分区备份、刷机日志或本地设备状态文件；每台设备都必须先做自己的备份。原始来源和版本见 [SOURCES.md](SOURCES.md)。

## 2. 安装驱动

1. 双击 `D:\lx04\drivers\UsbDk_1.0.22_x64.msi` 安装。
2. 右键 `D:\lx04\drivers\MTK\Mediatek-Drivers-Install.bat`，选择“以管理员身份运行”。
3. 重启电脑。打开 **命令提示符（cmd）**，输入：

```cmd
cd /d D:\lx04
py -3.12 mtkclient\mtk.py --help
```

能显示命令帮助后继续。下面的命令都在这个窗口执行，**每次只运行一条**。

## 3. 连接并备份

**每执行一条 `py -3.12 mtkclient\mtk.py` 命令，都要重新连接一次：**

1. 拔掉音箱 USB，等至少 5 秒，确认屏幕全黑。
2. 先在电脑运行命令，等出现 `Waiting for PreLoader VCOM`。
3. 按住音箱 **音量加**，插上电脑 USB；窗口开始滚动文字后松开。
4. 如提示重连，再按住音量加重插一次。等命令完成，再执行下一条。

先读分区表，确认能连接：

```cmd
py -3.12 mtkclient\mtk.py printgpt --noreconnect
```

再依次备份全部分区、两个引导区（第一条约需 2 小时）：

```cmd
py -3.12 mtkclient\mtk.py rl backup\partitions --noreconnect
```

```cmd
py -3.12 mtkclient\mtk.py r preloader backup\preloader_boot1.bin --parttype=boot1 --noreconnect
```

```cmd
py -3.12 mtkclient\mtk.py r preloader backup\preloader_boot2.bin --parttype=boot2 --noreconnect
```

检查没有读取失败，且 `backup\partitions` 中有 `proinfo.bin`（3 MiB）、`nvram.bin`（5 MiB）、`protect1.bin` 和 `protect2.bin`（各 10 MiB）；两个 `preloader_boot*.bin` 各 2 MiB。**把整个 `backup` 文件夹另存一份后再继续。只用自己设备的备份，不要上传到 GitHub。**

## 4. 刷入 X04G

按下面顺序执行。**每条都按第 3 步重新插线；读写报错就停下，不要接着刷。**

① 写分区表：

```cmd
py -3.12 mtkclient\mtk.py wf files\gpt_x04g.bin --noreconnect
```

② 写系统（约 10 分钟，等实际写入完成）：

```cmd
py -3.12 mtkclient\mtk.py wl global --preloader files\preloader_mico_x04g.bin --noreconnect
```

③ 写引导区：

```cmd
py -3.12 mtkclient\mtk.py w preloader files\boot1_for_microx04g.bin --parttype=boot1 --preloader files\preloader_mico_x04g.bin --noreconnect
```

④ 清除旧数据：

```cmd
py -3.12 mtkclient\mtk.py e userdata,cache,proinfo --noreconnect
```

⑤ 恢复自己的序列号信息：

```cmd
py -3.12 mtkclient\mtk.py w proinfo backup\partitions\proinfo.bin --noreconnect
```

⑥ 清除网络缓存：

```cmd
py -3.12 mtkclient\mtk.py e nvdata --noreconnect
```

⑦ 恢复自己的 Wi-Fi / 蓝牙信息：

```cmd
py -3.12 mtkclient\mtk.py w nvram backup\partitions\nvram.bin --noreconnect
```

⑧ 解锁引导：

```cmd
py -3.12 mtkclient\mtk.py da seccfg unlock --noreconnect
```

⑨ 重启：

```cmd
py -3.12 mtkclient\mtk.py reset --noreconnect
```

完成后拔掉电脑 USB，接 **5V / 2A 电源**，等待 10–15 分钟。出现 `Orange State` 正常。若进入 Recovery 提示无法加载系统，用音量键选择 `Wipe data/factory reset → Yes → Reboot system now`，静音键确认，再等开机。

## 5. 开启 ADB

确认能进入 X04G 界面后，拔电源，按第 3 步的方式分别执行：

```cmd
py -3.12 mtkclient\mtk.py w boot files\boot_global_magisk.bin --noreconnect
```

```cmd
py -3.12 mtkclient\mtk.py reset --noreconnect
```

开机后用数据线连电脑，执行：

```cmd
platform-tools\adb devices
platform-tools\adb shell id
```

设备状态为 `device` 且 `id` 显示 `uid=0(root)`，就说明 ADB root 可用。Magisk 管理界面和普通应用的 root 授权需要分别验证，见第 7 节。

若设备列表为空：在设备管理器中给音箱的 ADB 设备更新驱动，选择“浏览我的电脑以查找驱动程序”，目录选 `D:\lx04\drivers\Google`，然后重试。

## 6. 装桌面、键盘和返回键

依次执行：

```cmd
platform-tools\adb install apps\SimpleKeyboard.apk
platform-tools\adb shell ime enable rkr.simplekeyboard.inputmethod/.latin.LatinIME
platform-tools\adb shell ime set rkr.simplekeyboard.inputmethod/.latin.LatinIME
platform-tools\adb install apps\KISS.apk
platform-tools\adb shell monkey -p fr.neamar.kiss -c android.intent.category.LAUNCHER 1
platform-tools\adb shell am start -a android.settings.HOME_SETTINGS
```

在音箱上把 **KISS Launcher** 选为默认桌面；如弹出桌面选择框，选 KISS 和“始终”。按下面命令确认能回到 KISS，再关闭原闹钟界面：

```cmd
platform-tools\adb shell input keyevent 3
platform-tools\adb shell pm disable-user --user 0 com.google.android.apps.mediashell
platform-tools\adb shell settings put global device_provisioned 1
platform-tools\adb shell settings put secure user_setup_complete 1
```

安装返回键：

```cmd
platform-tools\adb install apps\VirtualSoftKeys.apk
platform-tools\adb shell appops set tw.com.daxia.virtualsoftkeys SYSTEM_ALERT_WINDOW allow
platform-tools\adb shell settings put secure enabled_accessibility_services tw.com.daxia.virtualsoftkeys/tw.com.daxia.virtualsoftkeys.service.ServiceFloating
platform-tools\adb shell settings put secure accessibility_enabled 1
platform-tools\adb shell monkey -p tw.com.daxia.virtualsoftkeys -c android.intent.category.LAUNCHER 1
```

在音箱上完成应用提示，从屏幕底边上滑即可调出导航。重启后若返回键消失，重新执行返回键设置中的 `appops`、两条 `settings` 和 `monkey` 命令。

最后打开 Wi-Fi 设置，在音箱上选网络、输入密码：

```cmd
platform-tools\adb shell am start -a android.settings.WIFI_SETTINGS
```

**到这里完成。** 可以从 KISS 打开应用，或用 `platform-tools\adb install 你的应用.apk` 安装其他应用。

## 7. 刷机后排查

以下命令在 Windows 的 `D:\lx04` 下执行。先用 `platform-tools\adb devices` 确认只连接了目标音箱，再用 `platform-tools\adb shell id` 确认是 `uid=0(root)`。已配置无线 ADB 的设备也可以用同样的方法检查；如果刷写后系统或无线 ADB 起不来，需要用 USB 和 MTK 工具恢复。

### 7.1 USB 连接卡住或开机循环

1. 一直显示 `Waiting for PreLoader VCOM` 时，检查是否使用数据线、是否在安装 UsbDk 后重启了电脑，以及是否先运行命令、再按住音量加插线。
2. 出现 `Wrong magic` 时，按 `Ctrl+C`，拔线等至少 5 秒，再按第 3 步连接。
3. 设备一开机就反复重启时，先换用 5V / 2A 电源。能稳定使用数小时、隔天常驻应用才退出的情况，按下一节查启动历史。

### 7.2 每天静默重启，常驻应用退出

1. 查运行时长和重启原因。日志时间按音箱时区显示；比较每天发生的时间时，先用 `date` 看设备时区。

```cmd
platform-tools\adb shell date
platform-tools\adb shell cat /proc/uptime
platform-tools\adb shell getprop sys.boot.reason
platform-tools\adb shell getprop persist.sys.boot.reason.history
platform-tools\adb shell dumpsys dropbox --print SYSTEM_LAST_KMSG
platform-tools\adb shell cmd jobscheduler get-job-state -u 0 com.google.assistant.core 1
```

2. 实机连续三天出现 `reboot,,quiescent`。内核日志显示 `system_server` 主动发出静默重启请求，设备上还查到 Google 助理的 `SilentRebootService` 每 30 分钟检查一次是否适合安排重启。先核实自己的设备也有相同日志和任务。可用 `platform-tools\adb shell dumpsys jobscheduler | findstr /i SilentRebootService` 确认任务是否属于 `com.google.assistant.core`、编号是否为 `1`。
3. 这版助理是设备所有者，直接停用组件会被系统拒绝。已验证的处理方式是使用服务自带的开关，在其 `files` 目录中创建空文件 `disable_silent_reboot`，让文件的属主、属组及 SELinux 上下文与应用目录一致，再取消已排程的任务。仅在已确认上一步的包名、任务编号和目录都匹配时执行。

```cmd
platform-tools\adb shell "set -e; d=/data/user/0/com.google.assistant.core/files; f=$d/disable_silent_reboot; test -d $d; touch $f; chown $(stat -c %u:%g $d) $f; chmod 600 $f; chcon $(ls -ldZ $d | awk '{print $5}') $f; ls -lZ $f"
platform-tools\adb shell cmd jobscheduler cancel -u 0 com.google.assistant.core 1
platform-tools\adb shell cmd jobscheduler get-job-state -u 0 com.google.assistant.core 1
```

4. 取消后若显示找不到任务，表示原排程已移除。实测禁用文件在手动重启后仍在；服务重新登记检查任务也有可能。最终以跨过平时重启时段后运行时间持续增长、启动历史没有新增 `reboot,,quiescent` 为准。现场记录时尚未跨过下一次静默重启时段。

### 7.3 ADB 有 root，Magisk 打不开或反复要求额外设置

1. 先分开检查 ADB root、Magisk 守护进程和管理应用。实测 `id` 返回 `uid=0(root)`，`magisk -v` 返回 `26.1:MAGISK:R`，而桌面上的 Magisk 是约 29 KiB 的 `1.0` 占位 APK。用官方完整的 [Magisk 26.1 APK](https://github.com/topjohnwu/Magisk/releases/tag/v26.1) 覆盖安装后，管理界面可以打开。其他版本应先核对守护进程版本。若确认需要替换占位包，将官网下载的 `Magisk-v26.1.apk` 放入 `D:\lx04`，运行 `platform-tools\adb install -r Magisk-v26.1.apk`。

```cmd
platform-tools\adb shell id
platform-tools\adb shell magisk -v
platform-tools\adb shell dumpsys package com.topjohnwu.magisk | findstr /i versionName
```

2. 如果每次打开都弹出 **Requires Additional Setup**，即使点 OK 重启也会再次出现，检查 Magisk 的 `preinit` 块设备。这台 X04G 的 Magisk 26.1 能识别出 `cache`，原启动镜像却没有写入对应配置。仅安装 APK 或再次重启无法补上启动镜像里的配置。

```cmd
platform-tools\adb shell magisk --preinit-device
platform-tools\adb shell "ls -l $(magisk --path)/.magisk/block/preinit"
```

3. 在这台设备上验证过的修复是先读取当前 `boot` 的完整备份并核对电脑与设备上的 SHA-256，使用同版本 `magiskboot` 解包当前镜像，在 ramdisk 的 `.backup/.magisk` 中补上 `PREINITDEVICE=cache`，再重新打包。写入前核对分区大小、内核、设备树和 ramdisk 中的其余内容；写入后读回比较一致再重启。只对 `magisk --preinit-device` 确实返回 `cache`、镜像与分区都已核对的设备使用此配置。保留当前设备的备份，并备好 USB 恢复方式。
4. 实测重启后无线 ADB 恢复，`preinit` 块设备存在，Magisk 首页显示已安装 26.1，且不再弹出额外设置提示。普通应用能否取得 Magisk root 仍需单独测试。本节是对已刷设备的修复记录，Release 中的原始镜像仍需按上述检查判断是否需要处理。

### 7.4 强制横屏与恢复自动旋转（可选）

音箱横放时，可将整台设备锁定在默认横向（旋转值 `0`）。这会让系统忽略应用请求的屏幕方向，对 B 站、Spotify 和其他应用都生效。两组命令都在 `D:\lx04` 的命令提示符中执行。

启用强制横屏：

```cmd
platform-tools\adb shell wm set-user-rotation lock 0
platform-tools\adb shell wm set-fix-to-user-rotation enabled
```

取消强制横屏并恢复自动旋转：

```cmd
platform-tools\adb shell wm set-fix-to-user-rotation disabled
platform-tools\adb shell wm set-user-rotation free
```

实测 B 站手机版的部分竖屏页面仍会在横屏中居中显示，两侧留黑边。屏幕方向和应用布局由不同设置控制，以上命令也不会修改 DPI。

### 7.5 Root 后停用预装更新组件（可选）

如果已经改用 KISS 等第三方桌面，并且不需要原厂自动更新，可以停用下面两个组件。先确认 ADB 已连接到目标音箱，且 `platform-tools\adb shell id` 返回 `uid=0(root)`。

```cmd
platform-tools\adb shell pm disable-user --user 0 com.google.assistant.display.android.updater
platform-tools\adb shell pm disable-user --user 0 com.mico.android.ota
platform-tools\adb shell pm list packages -d
```

- 上面两个包分别是 Google 助理显示更新器和小米 OTA。实测都显示为 `disabled-user`，小米 OTA 后台进程随之退出。停用后，这两个组件停止自动检查更新。预装 APK 仍在系统分区，能腾出的应用存储空间有限。
- 原厂主界面 `com.google.android.apps.mediashell` 的停用方法见第 6 步。先确认 KISS 能作为默认桌面打开。
- `com.google.assistant.core` 是这版系统的设备所有者，直接停用会被拒绝。对静默重启使用第 7.2 节的专用开关。实测保留了 Assistant Core、`com.google.assistant.oemapp` 和其他原厂组件。

需要重新启用这两个更新组件时，在同一个命令提示符执行：

```cmd
platform-tools\adb shell pm enable --user 0 com.google.assistant.display.android.updater
platform-tools\adb shell pm enable --user 0 com.mico.android.ota
```

### 7.6 清理后台与恢复可用内存（可选）

若按最近任务键没有反应，可以用 `platform-tools\adb shell dumpsys activity recents` 检查。实测系统显示 `mRecentsComponent=null`，没有配置最近任务界面。Android 仍会自动管理后台进程。感觉卡顿时，可以手动回收系统判定可结束的后台进程。

```cmd
platform-tools\adb shell grep MemAvailable /proc/meminfo
platform-tools\adb shell am kill-all
platform-tools\adb shell grep MemAvailable /proc/meminfo
```

`MemAvailable` 是清理前后的估算可用内存。这个操作不会清除账号、应用数据或文件。退出单个应用时也可以用系统应用详情页的「强行停止」，它会结束该应用的播放、下载及其他进程，直到再次打开。
