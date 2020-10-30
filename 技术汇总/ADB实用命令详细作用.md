## 										ADB实用命令详细作用

由于adb命令实在太多,网上搜索一大把,但是讲的都不是很详细,所以自己整理一份出来

​     **adb是什么?**：adb的全称为Android Debug Bridge，就是起到调试桥的作用。通过adb我们可以在Eclipse中方面通过DDMS来调试Android程序，说白了就是debug工具。adb的工作方式比较特殊，采用监听Socket TCP 5554等端口的方式让IDE和Qemu通讯，默认情况下adb会daemon相关的网络端口，所以当我们运行AndroidStudio时adb进程就会自动运行。

​    **adb有什么用?**：借助adb工具，我们可以管理设备或手机模拟器的状态。还可以进行很多手机操作，如安装软件、系统升级、运行shell命令等等。其实简而言说，adb就是连接Android手机与PC端的桥梁，可以让用户在电脑上对手机进行全面的操作!

### 常用命令:

- 1.显示当前运行的全部模拟器：  adb devices

- 2.启动ADB  adb start-server

- 3.停止ADB  adb kill-server

- 4.安装应用程序：    adb install -r [apk文件]

  -l #锁定该程序-r #重新安装该程序，保存数据-s #安装在SD卡内，而不是设备内部存储  例:adb install --r D:\mm.apk

- 5.卸载应用程序：    adb uninstall [packagename]
  例:adb uninstall com.tencent.mm
  -k #不删除程序运行所产生的数据和缓存目录(如软件的数据库文件)
  例:adb uninstall -k com.tencent.mm

- 6.讲手机设备中的文件copy到本地计算机：   
  adb pull 设备目录 本地目录例:adb pull /sdcard/mm.txt D:\(讲内存卡根目录的txt文件copy到D盘根目录)

- 7.将本地计算机的文件copy到手机设备中  adb push 本地目录 手机设备目录例:adb push D:\mm.txt /sdcard

- 8.查看adb命令帮助信息：    adb help

- 9.截屏例:adb shell screencap -p 截图文件路径

- 10.查看指定包名应用的数据库储存信息(包括储存的SQL语句) 例:adb shell dumpsys dbinfo[packagename]

- 11.查看指定的进程或则进程id的内存信息 
  adb shell dumpsys meminfo[packagename/pid]可以查看进程当前的内存情况

  例:adb shell dumpsys meminfo com.tencent.mm

- 12.查看指定包名应用的详细信息(相当于AndroidMainfest.xml中内容)
  adb shell dumpsys [packagename]例:adb shell dumpsys com.tencent.mm

- 13.查看当前应用的activity信息adb shell dumpsys activity top查看bug报告：
  adb bugreport

- 14.列出手机装的所有apk包名
  adb shell pm list packages
  系统应用:adb shell pm list packages -s
  第三方应用:adb shell pm list packages -3
  使用grep过滤 :adb shell pm list packages | grep qq

- 15.清除应用缓存信息:
  adb shell pm clear [packagename]

- 16.通过adb启动应用程序页面
  adb shell am start -n[包名+activity名]
  例:adb shell am start -n com.tencent.mm/.ui.SplashAcitvity
  强制停止应用有些时候应用卡死了，需要强制停止，则执行以下命令：adb shell am force-stop <packagename>// 如：adb shell am force-stop cn.androidstar.demo

- 17、记录无线通讯日志：  
  一般来说，无线通讯的日志非常多，在运行时没必要去记录，但我们还是可以通过命令，设置记录：
    adb shell
    logcat -b radio

- 18、获取设备的ID和序列号：   adb get-product
     adb get-serialno

- 19、访问数据库SQLite3  adb shell
     sqlite3#cd system/sd/data //进入系统内指定文件夹
  \#ls //列表显示当前文件夹内容
  \#rm -r xxx //删除名字为xxx的文件夹及其里面的所有文件
  \#rm xxx //删除文件xxx
  \#rmdir xxx //删除xxx的文件夹

- 20.导出设备信息
  adb get-serialno > 序列号.txt
  adb shell cat /sys/class/net/wlan0/address > MAC地址.txt
  adb shell getprop ro.product.model > 设备型号.txt
  adb shell getprop ro.build.version.release> 系统版本.txt
  adb shell pm list packages -s > 系统应用的所有包名.txt
  adb shell pm list packages -3 > 第三方应用包名.txt
  adb shell wm size > 屏幕分辨率.txt
  adb shell wm density > 屏幕密度.txt
  adb shell cat /proc/cpuinfo > CPU信息.txt
  adb shell pm list permissions -f > 权限.txt
  adb shell pm list users -f > 用户.txt

- 21.重启adb reboot

- 22.还有2个非常有用的命令:
  1.备份adb backup
  [-f <file>] [-apk|-noapk][-shared|-noshared] [-all] [-system|nosystem] [<packages...>]
  **例:adb backup -f mm.ab -noapk -noshared -nosystemcom.tencent.mm**你可以使用的最基本的命令是很简单的
  adb backup -all
  它将使用默认方式备份应用和设备的数据（不包含apk）到当前目录下并保存为文件backup.ab
  ※这个命令有可能不对每个设置都有效，如果你出现像这种 "adb: cannot open file ./backup.ab"的错误，使用 adb backup -all -fC:\backup.ab来代替，其中路径C:\可根据喜好替换
  对各个参数的解释：
  -f <file>
  用这个来选择备份文件存储在哪里，例如-f /backup/mybackup.ab将会使文件存储在根磁盘（Windows的C盘等等）下一个名为backup的文件夹里，并且备份文件名为mybackup.ab
  -apk|-noapk
  这个决定是否在备份里包含apk或者仅仅只备份应用数据，个人推荐使用-apk以免有的应用在应用市场找不到，如果不使用则默认的是-noapk
  -shared|-noshared
  这个参数用于决定是否备份设备共享的SD card内容，默认是-noshare，主要包括内部存储中的音乐、图片和视频，因此为保险起见，建议加上-share
  -all
  这个参数是一种简单地表达“所有应用”的说法，package参数可以选择备份单独的应用，如果你不是备份某个应用，使用-all备份整个系统
  -system|-nosystem
  这个参数决定-all标签是否包含系统应用，默认的是-system，根据情况可选择是否用-nosystem
  <packages...>
  如果你知道应用安装包的名称（例如com.google.android.apps.plus），就可以使用该参数备份特定应用。
  3.当决定如何执行备份后，输入你喜欢的命令，在华为G700上测试，使用命令
  adb backup -apk -all
  2.使用run-as在非root情况获取沙盒数据(前提是开启debuggable模式)
  \1.  shell@android:/data $ run-as com.your.package
  \2.  run-as com.your.package
  \3.  shell@android:/data/data/com.your.package $ cd /data/data/com.your.package 
  \4. cd /data/data/com.your.package
  \5. shell@android:/data/data/com.your.package $ ls 
  \6.  ls 
  \7.  cache 
  \8. databases
  \9. lib 
  \10. shared_prefs 
  \11. shell@android:/data/data/com.your.package $ cd databases 
  \12. cd databases 
  \13. shell@android:/data/data/com.your.package/databases $ ls 
  \14. yourpackagename.db 
  \15. $ cat preferences.db > /mnt/sdcard/yourpackagename.db  
  将你要访问的package目录下的db文件拷贝到sdcard中，这样就可以正常访问了！ 对文件进行增删
  法1:adb shell "run-aspackage.name chmod 666 /data/data/package.name/databases/file"
  adb pull /data/data/package.name/databases/file .
  adb shell "run-aspackage.name chmod 600 /data/data/package.name/databases/file"
  adb exec-out run-as package.name cat databases/file > file
  法2:> adb shellshell $ run-as com.example.packageshell $ chmod 666 databases/fileshell $ exit                              
  'run-as'shell $ cp /data/data/package.name/databases/file /sdcard/shell $ run-as com.example.packageshell $ chmod 600 databases/file> adb pull /sdcard/file .

  更新一些反编译常用命令:
  1.查看当前进程的内存的加载情况啊:
  cat /proc/7654/maps 查看当前进程内存的映射情况
  2.查看当前应用使用的端口号信息:
  cat /proc/[pid]/net/tcp
  3.查看进程的状态信息:
  cat /proc/[pid]/status可以通过该命令获取到当前进程的包名,PID,PPID等等重要信息(比较实用的命令)
  4.查看一个dex文件的详细信息
  dexdump [dex文件路径]
  5.使用aapt命令获取apk的清单文件
  aapt dump xmltree [apk包] [需要查看的资源文件xml]
  例:aapt dump xmltree mm.apk AndroidMainfest.xml > demo.txt(讲mm应用中的AndroidMainfest.xml文件导入到新建的demo.txt文本中)
  这里可能大家有个误区,aapt命令是与adb命令不是同一个命令,如果要使用和adb一样需要配置环境变量,也可以在SDK的build-tools文件夹内,shift+右键在此处打开命令窗口使用该命令!

























