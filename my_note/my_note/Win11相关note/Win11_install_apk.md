## 1.windows下载安装adb

### 下载adb

**Google很好的心，直接放出ADB的档案供人下载。下档路径如下：**

Windows版本：[https://dl.google.com/android/repository/platform-tools-latest-windows.zip](https://dl.google.com/android/repository/platform-tools-latest-windows.zip)
Mac版本：[https://dl.google.com/android/repository/platform-tools-latest-darwin.zip](https://dl.google.com/android/repository/platform-tools-latest-windows.zip)
Linux版本：[https://dl.google.com/android/repository/platform-tools-latest-linux.zip](https://dl.google.com/android/repository/platform-tools-latest-linux.zip)

> 将文件下载下来，解压缩到自定义的安装目录

### 配置环境变量

按键windows+ r打开运行，输入sysdm.cpl，回车。
高级》环境变量》系统变量》path
**将adb的存放路径添加进path中**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200831141605395.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3gyNTg0MTc5OTA5,size_16,color_FFFFFF,t_70#pic_center)

> 两次确定之后在重新打开命令行进行校验是否安装成功

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200831141800873.png#pic_center)

**显示版本信息代表安装成功**

## 2.Windows 11 WSA 安装 APK 方法：

1. 打开 WSA 安卓子系统设置页面，打开「 **开发人员模式** 」 选项
   ![WSA 开发人员模式](https://img.iplaysoft.com/wp-content/uploads/2021/win11-android-wsa/wsa_dev_mode.png!0x0.webp)
2. 记下上图设置项中显示出来的 WSA 的内部 IP 地址和端口号，如 `127.0.0.1:58526`
3. [下载安卓 ADB 命令行调试工具](https://www.iplaysoft.com/p/adb)，并参照文章教程，将 `adb` 命令加入到系统环境变量
4. 打开 Windows 终端 (命令行)，输入以下命令：

```bash

# 执行下面的命令能看到 adb 版本号则表示 ok

adb version

# 第 1 步：连接 WSA
adb connect 127.0.0.1:58526


# 第 2 步：安装 APK

adb install 你的APK文件完整路径
# 注意 .apk 的路径最好无中文且无空格，否则需要用英文双引号包裹。

#下面是例子：
adb install d:\download\apk\weixin.apk
adb install "d:\下载\异次元 iPlaySoft.com\qq.apk"


# 安装完成后，在 Windows 开始菜单的“所有应用”里就能找到你安装的 Android 应用
```

这样就能**使用 adb 命令安装 apk 文件到 Windows 11 安卓子系统** WSA 了。重点是开启开发者模式，获得正确 IP 地址以及正确安装 adb 命令 (环境变量)。

### (可选) 拖放 apk 文件安装的批处理脚本：

前面是手动输入命令安装 apk 的方法，如果你嫌麻烦，可以使用下面的批处理脚本，实现拖放 apk 文件安装的效果。方法是在桌面新建一个 `安装APK.txt` 的文本文档，拷贝粘贴下面的代码后保存。

```
@echo off
C:\文件夹路径\adb.exe connect 127.0.0.1:58526
C:\文件夹路径\adb.exe -s 127.0.0.1:58526 install %1
pause
```

* 其中 `C:\文件夹路径\adb.exe` 需要替换成你本机的 adb 所在路径
  (如果你已配置好环境变量，也可以只用 `adb` 代替)
* 而 `127.0.0.1:58526` 则要替换成你在 WSA 设置页面中看到的 IP 地址。
* 最后，将该文本文件的后缀名改成 `.bat` ，即重命名为：`安装APK.bat`。

**批处理脚本使用方法：**

直接将你想要安装的「.APK 文件」，鼠标拖住不放，然后移动到上「安装APK.bat」的脚本文件上，放手后即会自动进行安装。

### 成功在 Windows 11 上运行安卓 APP

经测试，很多常用的 Android 应用都能正常运行，而且流畅性很不错，[性能](https://www.iplaysoft.com/tag/%E6%80%A7%E8%83%BD)让人满意！秒杀众多[模拟器](https://www.iplaysoft.com/tag/%E6%A8%A1%E6%8B%9F%E5%99%A8)！而且安卓程序与 Win 11 之间的联动和融合的使用体验也做得非常好，甚至也能使用 Win11 的输入法直接在 APP 里打字，剪贴板也是互通的。

![Windows 11 安装安卓 APP](https://img.iplaysoft.com/wp-content/uploads/2021/win11-android-wsa/win11_android_wsa_apps.jpg!0x0.webp)

很多游戏也能运行，不过目前 WSA 暂未能调用硬件 GPU 加速，因此像原神等大作还很卡。另外，小部分 APP 会有闪退等兼容性问题，待日后[优化](https://www.iplaysoft.com/tag/%E4%BC%98%E5%8C%96)应该就完美了。

### 安装国内的 Android 应用商店：

当然，每次装软件都要用 adb 命令会比较麻烦，为了能更方便下载常用的安卓 APP，我们可在 WSA 里安装一个国内的应用商店，比如「[酷安应用市场](https://dl.iplaysoft.com/files/5598.html)」(其他的应用市场没试，估计也可以吧)，之后就能通过它快速地搜索、下载各种常用的 Android 应用和游戏了。

而且比较好的一点是，酷安似乎还能用来管理、卸载已安装好的 APP 程序。之后除了一些商店里没收录的 APP 还需要通过 apk 文件安装，其他基本都无需再使用命令行操作了。

## 3.adb命令-指定设备

若你的电脑连接多个设备（例：手机），而由于种种原因，你没有办法把这些设备拔掉（也可能是懒），但是你却想操作其中的一台设备你该如何做呢？

下面就教给你方法：

**1.使用方法举例**

![](https://images2018.cnblogs.com/blog/1191241/201804/1191241-20180410174208411-1723222298.png)

**2.具体详解**

### 命令语法

adb 命令的基本语法如下：

```
adb [-d|-e|-s ] 
```

如果只有一个设备/模拟器连接时，可以省略掉 `[-d|-e|-s ]` 这一部分，直接使用 `adb `。

### 为命令指定目标设备

如果有多个设备/模拟器连接，则需要为命令指定目标设备。

| 参数   | 含义                                               |
| ------ | -------------------------------------------------- |
| -d     | 指定当前唯一通过 USB 连接的 Android 设备为命令目标 |
| -e     | 指定当前唯一运行的模拟器为命令目标                 |
| `-s` | 指定相应 serialNumber 号的设备/模拟器为命令目标    |

## 4.具体实现代码

```c
C:\Users\24962>adb devices
List of devices attached
192.168.157.134:5555    device
emulator-5554   device


C:\Users\24962>adb -s emulator-5554 192.168.157.134
adb.exe: unknown command 192.168.157.134

C:\Users\24962>adb -s emulator-5554 connect 192.168.157.134
already connected to 192.168.157.134:5555

C:\Users\24962>adb -s emulator-5554 install "C:\Users\24962\Desktop\base.apk"
Performing Streamed Install
Success

C:\Users\24962>adb -s emulator-5554 install "C:\Users\24962\Desktop\base(1).apk"
Performing Streamed Install
Success

C:\Users\24962>

```
