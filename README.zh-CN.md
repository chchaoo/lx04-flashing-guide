<div align="center">

<img src="docs/images/logo.svg" width="96" height="96" alt="">

# LX04 刷机指南

**把小爱触屏音箱，变成可自由安装应用的迷你屏幕。**

LX04 → X04G · 备份、刷机、桌面与导航，一步一步完成。

[![Device](https://img.shields.io/badge/device-LX04_%E2%86%92_X04G-2f81f7)](#开始之前)
[![Host](https://img.shields.io/badge/host-Windows-0078d6)](#1-准备工具与文件)
[![Guide](https://img.shields.io/badge/type-step--by--step-2ea44f)](#操作路线)

[English](README.md) · **简体中文**

</div>

---

## 操作路线

<p align="center"><img src="docs/images/steps-zh.svg" width="900" alt="准备 → 备份 → 刷机 → 桌面与导航"></p>

教程依据一台国行 **LX04** 成功转换为 **X04G（Android Go 10）** 的操作记录整理。电脑使用 Windows，命令均在 **命令提示符（cmd）** 中执行，逐条运行。完成后可安装普通 Android APK。

> [!WARNING]
> 刷机会清除设备数据，并有无法启动的风险。开始前必须备份自己的设备，尤其是 `proinfo.bin` 和 `nvram.bin`。不要使用他人的设备备份，也不要重新锁定 bootloader。

## 开始之前

- 确认机身型号为 **LX04**；其他小爱音箱型号不适用。
- 准备 Windows 实体电脑、支持数据传输的 Micro-USB 线、稳定的 **5V / 2A 电源**。
- 电脑留出至少 **15 GB** 空间，备份再另存一份。
- 全程使用下文的 `D:\lx04` 和 `D:\platform-tools`。没有 D 盘时，统一替换盘符；路径避免中文和空格。
- 本仓库提供教程，固件、驱动和 APK 从下表原始来源下载。

## 1. 准备工具与文件

| 工具或文件 | 下载来源 | 放置 / 操作 |
|---|---|---|
| Python 3.12、Git、7-Zip | [Python](https://www.python.org/downloads/windows/) · [Git](https://git-scm.com/downloads/win) · [7-Zip](https://www.7-zip.org/) | 安装 Python 时勾选 **Add python.exe to PATH** |
| mtkclient | [bkerler/mtkclient](https://github.com/bkerler/mtkclient) | 按下面命令安装 |
| UsbDk x64 | [官方 Releases](https://github.com/daynix/UsbDk/releases) | 原记录使用 `UsbDk_1.0.22_x64.msi`；安装后重启电脑 |
| `xiaomi-x04g-lx04-main.zip` | [AyraHikari/xiaomi-x04g-lx04](https://github.com/AyraHikari/xiaomi-x04g-lx04) → Code → Download ZIP | 解压到 `D:\lx04\repo\` |
| `dump_mico_x04g.tar.xz` | [X04G 固件归档](https://archive.org/details/dump_mico_x04g) | 用 7-Zip 解压两次，将分区 `.bin` 文件直接放入 `D:\lx04\global\` |
| SDK Platform-Tools | [Android 官方下载](https://developer.android.com/tools/releases/platform-tools) | 解压后使 `adb.exe` 位于 `D:\platform-tools\` |
| KISS Launcher APK | [F-Droid](https://f-droid.org/packages/fr.neamar.kiss/) | 页面点击 **Download APK**，保存为 `D:\lx04\apps\KISS.apk` |
| Simple Keyboard APK | [F-Droid](https://f-droid.org/packages/rkr.simplekeyboard.inputmethod/) | 保存为 `D:\lx04\apps\SimpleKeyboard.apk` |
| VirtualSoftKeys APK | [F-Droid](https://f-droid.org/en/packages/tw.com.daxia.virtualsoftkeys/) | 下载版本 `0.2.0.180212.1 (10)`，保存为 `D:\lx04\apps\VirtualSoftKeys.apk` |

APK 文件名是本教程约定的名称，下载后重命名即可；无需安装 F-Droid 客户端。选择兼容 Android 10 的 APK；含原生库时先用第 5 步命令确认 ABI。

在 cmd 中建目录并安装 mtkclient：

```cmd
mkdir D:\lx04\backup\partitions D:\lx04\global D:\lx04\repo D:\lx04\tools D:\lx04\apps
cd /d D:\lx04
git clone https://github.com/bkerler/mtkclient
cd /d D:\lx04\mtkclient
git checkout cd25cf9c1ff6d36e82697ac2c798e69e9cfb78c3
python -m pip install -r requirements.txt
python -m pip install .
```

这里固定为本地成功记录所用的 mtkclient 提交，避免新版命令变化。如果安装提示缺少 Visual C++，安装 [Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/) 的“使用 C++ 的桌面开发”，再重试。

将 `repo\xiaomi-x04g-lx04-main\files\FlashTool.7z` 解压到 `D:\lx04\tools\`，找到 `driver\Mediatek-Drivers-Install.bat`，右键以管理员身份运行，安装 MTK VCOM 驱动。**本教程刷写使用 mtkclient。**

刷机前确认目录：

```text
D:\lx04\
├── mtkclient\mtk.py
├── backup\partitions\              ← 下一步生成自己的备份
├── global\                         ← 固件分区直接放在这里
│   ├── gpt.bin
│   ├── boot.bin
│   ├── super.bin
│   └── ...                         ← 其余解压出的分区 .bin
├── repo\xiaomi-x04g-lx04-main\files\
│   ├── gpt_x04g.bin
│   ├── preloader_mico_x04g.bin
│   ├── boot1_for_microx04g.bin
│   └── boot_global_magisk.bin
└── apps\
    ├── KISS.apk
    ├── SimpleKeyboard.apk
    └── VirtualSoftKeys.apk

D:\platform-tools\adb.exe
```

固件归档内的 `sha256sums.txt` 可用于核验分区文件。例如运行 `certutil -hashfile D:\lx04\global\super.bin SHA256`，与清单对应行比较。不把额外固件、自己的备份或 APK 混入 `global`；`wl` 会递归扫描该目录。

## 2. 连接并备份自己的设备

### 每条 mtkclient 命令都要重新连接

1. 拔掉设备 USB，等至少 **5 秒**，确认屏幕全黑。
2. **先执行电脑上的命令**，等待 `Waiting for PreLoader VCOM`。
3. 按住设备的 **音量加键**，保持按住并接入电脑 USB。
4. 窗口开始握手、滚动文字后松开。若要求重连，再按住音量加重插一次。
5. 等该命令完成，再为下一条重复上述动作。

正常连接会出现 `Device detected`、`Kamakiri Run`、`Boot to succeeded`。遇到 `Wrong magic`，按 `Ctrl+C` 停止，断电后重来。

先测试连接，读出分区表：

```cmd
cd /d D:\lx04\mtkclient
python mtk.py printgpt --noreconnect
```

按上面的动作重新连接，逐条备份全部分区及两个引导区：

```cmd
python mtk.py rl D:\lx04\backup\partitions --noreconnect
```

```cmd
python mtk.py r preloader D:\lx04\backup\preloader_boot1.bin --parttype=boot1 --noreconnect
```

```cmd
python mtk.py r preloader D:\lx04\backup\preloader_boot2.bin --parttype=boot2 --noreconnect
```

分区备份可能需要约两小时。检查输出无读失败，并确认下列文件存在、非空；原记录参考大小如下：

| 自己的备份 | 参考大小 | 用途 |
|---|---:|---|
| `backup\partitions\proinfo.bin` | 3 MiB | 保留设备序列号 |
| `backup\partitions\nvram.bin` | 5 MiB | 保留 Wi-Fi / 蓝牙标识与校准数据 |
| `backup\partitions\protect1.bin`、`protect2.bin` | 各 10 MiB | 原系统设备数据 |
| `backup\preloader_boot1.bin`、`preloader_boot2.bin` | 各 2 MiB | 原引导区 |

**备份整个 `backup` 文件夹到另一个位置再继续。** `proinfo` 或 `nvram` 缺失时不要刷机；备份包含设备唯一信息，不要上传公开仓库。

## 3. 刷入 X04G

在 `D:\lx04\mtkclient` 中按顺序执行。**每条都重新进行第 2 步的连接动作**；出现实际分区写失败时先停止排查，不继续叠加操作。

**① 写 X04G 分区表** — `gpt_x04g.bin`

```cmd
python mtk.py wf D:\lx04\repo\xiaomi-x04g-lx04-main\files\gpt_x04g.bin --noreconnect
```

**② 写 X04G 固件分区** — `global` 文件夹；本步时间较长。

```cmd
python mtk.py wl D:\lx04\global --preloader D:\lx04\repo\xiaomi-x04g-lx04-main\files\preloader_mico_x04g.bin --noreconnect
```

归档的 `.state`、校验清单或 `full_flash` 等非分区项目可能提示找不到同名分区并跳过；这与实际的 `boot`、`super` 等分区写入失败不同。确认系统分区均成功写入再继续。

**③ 写 boot1 引导区** — `boot1_for_microx04g.bin`

```cmd
python mtk.py w preloader D:\lx04\repo\xiaomi-x04g-lx04-main\files\boot1_for_microx04g.bin --parttype=boot1 --preloader D:\lx04\repo\xiaomi-x04g-lx04-main\files\preloader_mico_x04g.bin --noreconnect
```

**④ 清除旧用户数据、缓存及固件附带的设备信息**

```cmd
python mtk.py e userdata,cache,proinfo --noreconnect
```

**⑤ 写回自己的设备信息** — 第 2 步备份的 `proinfo.bin`

```cmd
python mtk.py w proinfo D:\lx04\backup\partitions\proinfo.bin --noreconnect
```

**⑥ 清除 nvdata**

```cmd
python mtk.py e nvdata --noreconnect
```

**⑦ 写回自己的 nvram** — 第 2 步备份的 `nvram.bin`

```cmd
python mtk.py w nvram D:\lx04\backup\partitions\nvram.bin --noreconnect
```

**⑧ 解锁 bootloader** — 必须在整机固件写入之后执行。

```cmd
python mtk.py da seccfg unlock --noreconnect
```

**⑨ 重启**

```cmd
python mtk.py reset --noreconnect
```

提示 reset 完成后拔线，改接 **5V / 2A 电源**开机，首次等 **10–15 分钟**。出现 **Orange State** 是解锁后的正常提示。

若进入 Android Recovery 并提示无法加载系统，音量键选择 **Wipe data/factory reset → Yes → Reboot system now**，静音键确认，然后再等待启动。

## 4. 启用 ADB root

确认已能进入 X04G 界面后，再用 mtkclient 写入带 Magisk 的 boot。两条命令仍各自重新连接：

```cmd
python mtk.py w boot D:\lx04\repo\xiaomi-x04g-lx04-main\files\boot_global_magisk.bin --noreconnect
```

```cmd
python mtk.py reset --noreconnect
```

启动后用数据线连接电脑，在 cmd 执行：

```cmd
cd /d D:\platform-tools
adb devices
adb shell id
adb shell getprop ro.product.cpu.abilist
```

设备状态为 `device`，且 `id` 输出含 `uid=0(root)` 即成功。若电脑有多个 Android 设备，先断开其他设备。本 boot 在原记录中提供 **ADB root**，Magisk 守护进程不可用，应用无法通过 Magisk 申请 root；下面命令直接执行，不加 `su -c`。

## 5. 装好键盘、桌面和导航键

以下命令都在 `D:\platform-tools` 执行。

### 键盘：Simple Keyboard

系统原厂没有可用输入法，先装键盘，便于填写 Wi-Fi 等信息：

```cmd
adb install D:\lx04\apps\SimpleKeyboard.apk
adb shell ime enable rkr.simplekeyboard.inputmethod/.latin.LatinIME
adb shell ime set rkr.simplekeyboard.inputmethod/.latin.LatinIME
```

### 桌面：KISS Launcher

```cmd
adb install D:\lx04\apps\KISS.apk
adb shell monkey -p fr.neamar.kiss -c android.intent.category.LAUNCHER 1
adb shell am start -a android.settings.HOME_SETTINGS
```

在默认主屏幕设置里选择 **KISS Launcher**。如果出现“使用哪个应用作为主屏幕”，选 KISS 并点“始终”。按下面命令验证主页能回到 KISS：

```cmd
adb shell input keyevent 3
```

如果主屏幕设置不显示 KISS，可先解析它的 HOME 组件，再设置。组件名以设备实际输出为准：

```cmd
adb shell cmd package resolve-activity --brief -a android.intent.action.MAIN -c android.intent.category.HOME -p fr.neamar.kiss
```

复制输出末尾的 `fr.neamar.kiss/...`，替换下面的 `HOME_COMPONENT`：

```cmd
adb shell cmd package set-home-activity HOME_COMPONENT
adb shell input keyevent 3
```

确认 KISS 可用后，关闭原闹钟界面及配网提示：

```cmd
adb shell pm disable-user --user 0 com.google.android.apps.mediashell
adb shell settings put global device_provisioned 1
adb shell settings put secure user_setup_complete 1
```

需恢复原界面时运行 `adb shell pm enable com.google.android.apps.mediashell`。

### 导航键：VirtualSoftKeys

系统没有常规返回 / 主页导航栏，安装悬浮导航键：

```cmd
adb install D:\lx04\apps\VirtualSoftKeys.apk
adb shell appops set tw.com.daxia.virtualsoftkeys SYSTEM_ALERT_WINDOW allow
adb shell am start -a android.settings.ACCESSIBILITY_SETTINGS
```

在无障碍设置中打开 **VirtualSoftKeys** 服务，返回应用完成提示设置；从屏幕底边上滑显示导航栏。若设置页不方便操作，可用下面两条命令启用（会覆盖已有的无障碍服务列表；新刷的设备可直接使用）：

```cmd
adb shell settings put secure enabled_accessibility_services tw.com.daxia.virtualsoftkeys/tw.com.daxia.virtualsoftkeys.service.ServiceFloating
adb shell settings put secure accessibility_enabled 1
adb shell monkey -p tw.com.daxia.virtualsoftkeys -c android.intent.category.LAUNCHER 1
```

**重启后导航键可能失效**：重新启用无障碍服务，再打开应用；也可重跑上面三条命令。临时返回 / 主页分别用 `adb shell input keyevent 4`、`adb shell input keyevent 3`。

## 6. Wi-Fi、无线 ADB 与显示

### 连 Wi-Fi

```cmd
adb shell am start -a android.settings.WIFI_SETTINGS
```

在屏幕上选择网络并用键盘输入密码。查看分配到的 IP：

```cmd
adb shell ip addr show wlan0
```

### 开启无线 ADB

USB 连接时执行以下两条。第二条执行后连接断开属于正常现象：

```cmd
adb shell setprop persist.adb.tcp.port 5555
adb shell setprop ctl.restart adbd
```

之后接回稳定电源，电脑与设备在同一局域网。将下文 `DEVICE_IP` 替换为设备实际 IP：

```cmd
adb connect DEVICE_IP:5555
adb devices
```

此设置重启后保留。这套镜像的无线 ADB 相当于局域网 root 入口，仅在可信网络启用，不向互联网开放 5555 端口。关闭方法：

```cmd
adb shell setprop persist.adb.tcp.port -1
adb shell setprop ctl.restart adbd
```

### 分辨率、显示密度与方向

原屏幕为 **800 × 480**。先查询，再按需修改；**密度越小，界面能显示的内容越多**，通常先调密度即可。

```cmd
adb shell wm size
adb shell wm density
adb shell wm density 140
adb shell settings put system accelerometer_rotation 0
adb shell settings put system user_rotation 0
```

需要指定逻辑分辨率时，例如设为原屏幕尺寸：

```cmd
adb shell wm size 800x480
```

恢复默认分辨率 / 密度：

```cmd
adb shell wm size reset
adb shell wm density reset
```

`user_rotation` 可取 `0 / 1 / 2 / 3`，对应原方向及旋转 90° / 180° / 270°；应用仍可能主动请求其他方向。

如果时间不对，开启网络时间并设置自己的时区（示例为中国标准时间）：

```cmd
adb shell settings put global auto_time 1
adb shell settings put global auto_time_zone 0
adb shell setprop persist.sys.timezone Asia/Shanghai
adb shell date
```

## 完成：像小电视一样使用

- **AirPlay 投屏**：安装 [AirPlay Server](https://github.com/jqssun/android-airplay-server)，启动接收服务；iPhone 与设备接入同一网络，在“控制中心 → 屏幕镜像”选择它。
- **IPTV**：安装 [OwnTV](https://github.com/ahXN00/OwnTV)，添加 M3U 播放列表。可从 [iptv-org/iptv](https://github.com/iptv-org/iptv) 选择国家 / 分类列表，例如 `https://iptv-org.github.io/iptv/index.m3u`。全部频道列表较大，小内存设备优先用较小的分类列表。
- **iPhone 操作**：使用 [atvTools](https://atvtools.app/)（[App Store](https://apps.apple.com/app/atvtools/id1661419573)），通过同一局域网的设备 IP 和 ADB 端口 **5555** 连接，可遥控、输入文字、安装 APK、运行 ADB shell 及查看设备屏幕镜像。这里的屏幕镜像是**在手机上查看设备画面**；iPhone 投到小爱屏幕使用上面的 AirPlay Server。

这些是刷机后的可选用途；应用和具体媒体格式在 LX04 上的兼容性、流畅度以实际版本为准，原始刷机记录未覆盖这些应用的验证。Android 应用安装通用命令：`adb install D:\lx04\apps\应用.apk`。

## 遇到问题

| 现象 | 处理 |
|---|---|
| 一直等设备 | 确认是数据线；UsbDk 安装后已重启；先运行命令，再按音量加插线；换电脑直连 USB 口 |
| `Wrong magic` | `Ctrl+C` 停止，拔线断电后按标准动作重新连接 |
| 首次开机慢 / 重启 | 使用 5V / 2A 电源，等 10–15 分钟；Recovery 提示无法加载系统时清除数据 |
| `adb devices` 没设备 | 确认已写入第 4 步 boot，数据线正常；检查设备管理器 ADB 驱动，可参考 [Google USB Driver](https://developer.android.com/studio/run/win-usb) |
| APK 安装报版本 / ABI 错误 | 下载兼容 Android 10 且 ABI 匹配的独立 APK；用 `adb shell getprop ro.product.cpu.abilist` 查询 |
| 重启后没有返回键 | 重新开启 VirtualSoftKeys 无障碍服务，启动应用 |
| 无线 ADB 连不上 | 重新检查设备 IP、Wi-Fi 和 5555 端口；必要时 USB 重连配置 |

## 来源与许可

刷机方法参考 [AyraHikari/xiaomi-x04g-lx04](https://github.com/AyraHikari/xiaomi-x04g-lx04)，并根据本机成功记录整理；工具来自 [bkerler/mtkclient](https://github.com/bkerler/mtkclient)。本教程不重新分发固件、驱动、APK 或设备备份。仓库自有文档与图标采用 [MIT 许可](LICENSE)，外部项目遵循各自许可。
