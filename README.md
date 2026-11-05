# 小米小爱触屏音箱 LX04 刷机

把国行 **LX04** 刷成 **X04G（Android 10）**，再装好桌面、键盘和返回键。仅适用于机身标签为 LX04 的设备。

需要 Windows 10/11 **64 位**电脑、Micro-USB **数据线**、5V / 2A 电源和至少 15 GB 空间。刷机会清空数据，先完成第 3 步备份。

## 1. 下载原始文件并准备目录

打开 [v1.0.0 Release](https://github.com/chchaoo/lx04-flashing-guide/releases/latest)，下载下面的附件。它们分别提供，不是重新制作的整合刷机包。

| 附件 | 放置方法 |
|---|---|
| `python-3.12.10-amd64.exe` | 安装 Python 3.12，保留 Python Launcher（`py`） |
| `mtkclient-cd25cf9.zip` | 解压，将源码目录重命名为 `C:\lx04\mtkclient`，其中应有 `mtk.py` |
| `xiaomi-x04g-lx04-main.zip` | 将其 `files` 目录放到 `C:\lx04\files` |
| `dump_mico_x04g.tar.xz` | 解压到 `C:\lx04\global`，这一层直接包含 `boot.bin` 等分区文件 |
| `platform-tools_r37.0.1-windows.zip` | 解压得到 `C:\lx04\platform-tools\adb.exe` |
| `UsbDk_1.0.22_x64.msi` | 放到 `C:\lx04\drivers` |
| `usb_driver_r13-windows.zip` | 将其中 `usb_driver` 目录重命名为 `C:\lx04\drivers\Google` |
| `KISS.apk`、`SimpleKeyboard.apk`、`VirtualSoftKeys.apk` | 放到 `C:\lx04\apps` |

另外解开 `C:\lx04\files\FlashTool.7z`，将其中 `FlashTool\driver` 目录的内容放到 `C:\lx04\drivers\MTK`，应能看到 `Mediatek-Drivers-Install.bat`。使用能够解开 ZIP、7z 和 tar.xz 的解压工具。不要把固件再多套一层目录。

打开 **命令提示符（cmd）**，准备目录并安装命令行依赖：

```cmd
cd /d C:\lx04
mkdir backup
py -3.12 -m pip install pyusb pycryptodome pycryptodomex colorama pyserial mfusepy keystone-engine capstone unicorn
py -3.12 mtkclient\mtk.py --help
```

能显示帮助后继续。后文都在 `C:\lx04` 下执行，路径不要有中文或空格。附件不包含任何个人分区备份、刷机日志或本地设备状态文件；每台设备都必须先做自己的备份。原始来源和版本见 [SOURCES.md](SOURCES.md)。

## 2. 安装驱动

1. 双击 `C:\lx04\drivers\UsbDk_1.0.22_x64.msi` 安装。
2. 右键 `C:\lx04\drivers\MTK\Mediatek-Drivers-Install.bat`，选择“以管理员身份运行”。
3. 重启电脑。打开 **命令提示符（cmd）**，输入：

```cmd
cd /d C:\lx04
py -3.12 mtkclient\mtk.py --help
```

能显示命令帮助后继续。下面的命令都在这个窗口执行，**每次只运行一条**。

## 3. 连接并备份

**每执行一条 `mtk` 命令，都要重新连接一次：**

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

显示 `device` 和 `uid=0(root)` 即成功。此镜像提供 ADB root，应用不能通过 Magisk 申请 root。

若设备列表为空：在设备管理器中给音箱的 ADB 设备更新驱动，选择“浏览我的电脑以查找驱动程序”，目录选 `C:\lx04\drivers\Google`，然后重试。

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

连接时一直等待：确认是数据线、UsbDk 装完已重启，并且先运行命令再按音量加插线。出现 `Wrong magic`：按 `Ctrl+C`，拔线等 5 秒后重来。开机循环重启：换 5V / 2A 电源。
