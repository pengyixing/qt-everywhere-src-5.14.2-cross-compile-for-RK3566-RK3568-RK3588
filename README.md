# This is qt5.14.2 cross-compile package for HYY RK3566 RK3568 RK3588 device with Ubuntu OS

This is the documentation for RK3566 RK3568 RK3588 10.1inch 10.6inch 14inch 15.6inch Tablet products, written by RSD Team of HYY Technology Co.,Ltd.

# How we config qt5.14.2
see config file [here](auto_config.sh)
```
#!/bin/bash

../qt-everywhere-src-5.14.2/configure \
	-xplatform linux-arm-som-rk3568 \
	-prefix /media/admin_/RK3568SDK/shadow_build_qt_5.14.2/sysbase/qt5.14.2_install \
	-sysroot /media/admin_/RK3568SDK/rk3588_sysroot \
	-I ~/sysroot/usr/include \
	-L ~/sysroot/usr/lib \
	-L ~/sysroot/usr/lib/aarch64-linux-gnu \
	-L ~/sysroot/lib \
	-release -opensource -confirm-license \
	-opengl es2 \
	-gstreamer 1.0 \
	-skip qtmacextras  -skip qtandroidextras -skip qtlocation -skip qtscxml -skip qtxmlpatterns \
	-skip qtactiveqt -skip qtsensors -skip qtconnectivity -skip webview -skip qtandroidextras \
	-skip qtwebchannel -skip qtpurchasing -skip qtwebglplugin -skip qtwebengine -skip qtremoteobjects \
	-skip qtspeech -skip qtnetworkauth -skip qt3d -skip qtcharts -skip qtgamepad -skip qtdoc \
	-skip qttools -skip qtdatavis3d -skip qtlottie -skip qtmacextras -skip qtquick3d -skip qtwayland -skip qtwinextras \
	-nomake tests -nomake examples \
	-make libs \
	-pch \
	-qt-libjpeg \
	-qt-libpng \
	-qt-zlib \
	-no-sse2 \
	-no-openssl \
	-no-cups \
	-no-glib \
	-no-separate-debug-info \
```
# How we config build support gstreamer 1.0
1. install build tools on target board [[RK3588 Mainboard (SBC)](https://github.com/pengyixing/RK3588-Development-Board)]
```
sudo apt-get install g++-aarch64-linux-gnu on target board
```

2. Downloard QT qt-everywhere-src-5.14.2.tar.xz and uncompress
```
tar xvf qt-everywhere-src-5.14.2.tar.xz
```
3. install gstreamer packages on target board [[RK3588 Mainboard (SBC)](https://github.com/pengyixing/RK3588-Development-Board)]
```
sudo apt-get install libgstreamer1.0-0 gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-doc gstreamer1.0-tools
sudo apt-get install libunwind-dev
```
4. Check gstreamer installed on target board [[RK3588 Mainboard (SBC)](https://github.com/pengyixing/RK3588-Development-Board)]
```
pkg-config gstreamer-1.0 --cflags
-pthread -I/usr/include/gstreamer-1.0 -I/usr/include/aarch64-linux-gnu -I/usr/include/glib-2.0 -I/usr/lib/aarch64-linux-gnu/glib-2.0/include
```
```
pkg-config gstreamer-1.0 --libs
-lgstreamer-1.0 -lgobject-2.0 -lglib-2.0
```
5. gstreamer 1.0 will failed if have no libgstreamer-plugins-base1.0-dev when do configure check on target board [[RK3588 Mainboard (SBC)](https://github.com/pengyixing/RK3588-Development-Board)]
``` 
sudo apt install libgstreamer-plugins-base1.0-dev
```
6. How we config build support ALSA
```
sudo apt-get install libasound2-dev
```
7. Do config check on target board [[RK3588 Mainboard (SBC)](https://github.com/pengyixing/RK3588-Development-Board)] as below:
``` 
./configure -release -opensource -confirm-license -gstreamer 1.0
```
will show as below:
``` 
Qt Multimedia:
  ALSA ................................... yes
  GStreamer 1.0 .......................... yes
  GStreamer 0.10 ......................... no
  Video for Linux ........................ yes
  OpenAL ................................. no
  PulseAudio ............................. no
  Resource Policy (libresourceqt5) ....... no
  Windows Audio Services ................. no
  DirectShow ............................. no
  Windows Media Foundation ............... no
```
8. Sync sysroot from target board [[RK3588 Mainboard (SBC)](https://github.com/pengyixing/RK3588-Development-Board)] to host for cross-compile
``` 
mkdir sysroot sysroot/usr sysroot/opt
rsync -avz hyy@hyytarget:/lib sysroot
rsync -avz hyy@hyytarget:/usr/include sysroot/usr
rsync -avz hyy@hyytarget:/usr/lib sysroot/usr
```
9. How we config qmake.conf on Host
```
cd /media/admin_/RK3568SDK/shadow_build_qt_5.14.2/qtbase/mkspecs/
mkdir linux-arm-som-rk3568
cp linux-aarch64-gnu-g++/* linux-arm-som-rk3568/
```
Modefied qmake.conf
```
#
# qmake configuration for building with aarch64-linux-gnu-g++
#

MAKEFILE_GENERATOR      = UNIX
CONFIG                 += incremental
QMAKE_INCREMENTAL_STYLE = sublib

QT_QPA_DEFAULT_PLATFORM = linuxfb
QMAKE_CFLAGS_RELEASE += -O2 -march=armv8-a -lts
QMAKE_CXXFLAGS_RELEASE += -O2 -march=armv8-a -lts

include(../common/linux.conf)
include(../common/gcc-base-unix.conf)
include(../common/g++-unix.conf)

QMAKE_INCDIR_OPENGL_ES2 = /media/admin_/RK3568SDK/openGL/mesa-12.0.5/install/include
QMAKE_LIBDIR_OPENGL_ES2 = /media/admin_/RK3568SDK/openGL/mesa-12.0.5/install/lib
QMAKE_LIBS_OPENGL_ES2 = -lglapi -lGLESv2

QMAKE_INCDIR = $$[QT_SYSROOT]/usr/include                             #指定sysroot头文件
QMAKE_LIBDIR = $$[QT_SYSROOT]/usr/lib                                 #指定sysroot库文件
QMAKE_LIBDIR += $$[QT_SYSROOT]/usr/lib/aarch64-linux-gnu              #指定sysroot库文件
QMAKE_LIBDIR += $$[QT_SYSROOT]/lib                                    #指定sysroot库文件

QMAKE_CXXFLAGS += -ludev -lffi -std=c++11                                 #链接选项
QMAKE_LFLAGS += -Wl,-rpath-link,$$[QT_SYSROOT]/usr/lib                    #链接sysroot库文件
QMAKE_LFLAGS += -Wl,-rpath-link $$[QT_SYSROOT]/usr/lib/aarch64-linux-gnu  #链接sysroot库文件
QMAKE_LFLAGS += -Wl,-rpath-link $$[QT_SYSROOT]/lib                        #链接sysroot库文件



# modifications to g++.conf
QMAKE_CC                = aarch64-linux-gnu-gcc
QMAKE_CXX               = aarch64-linux-gnu-g++
QMAKE_LINK              = aarch64-linux-gnu-g++
QMAKE_LINK_SHLIB        = aarch64-linux-gnu-g++

# modifications to linux.conf
QMAKE_AR                = aarch64-linux-gnu-ar cqs
QMAKE_OBJCOPY           = aarch64-linux-gnu-objcopy
QMAKE_NM                = aarch64-linux-gnu-nm -P
QMAKE_STRIP             = aarch64-linux-gnu-strip
load(qt_config)
```

# RK3588 mainboard info for this documentation
- [RK3588 Mainboard (SBC) Documents](https://github.com/pengyixing/RK3588-Development-Board)



## Get More technical Support

###### RK3588 Development Board

\- [RK3588 Development Board](https://github.com/industrialtablet/RK3588-Development-Board)

###### ota upgrade tools(otaStar) and server

\- [RK3566/RK3568/RK3588 Android OTA upgrade tools and server](https://github.com/tablet-pc/otastar)

###### How Qt5.14.2 cross-compile

\- [RK3588 Qt5.14.2 cross-compile for Ubuntu and Debian Linux OS](https://github.com/pengyixing/qt-everywhere-src-5.14.2-cross-compile-for-RK3566-RK3568-RK3588)

Build Videorecorder Bundle use Networkoptix Client on HYY H-3588 Tablet

\- [Build Videorecorder Bundle use Networkoptix Client](https://github.com/industrialtablet/Build-Videorecorder-Bundle-use-Networkoptix-Client-on-HYY-RK3566-Tablet)

Build new ubuntu rootfs for HYY H-3588 Tablet

-[Build new ubuntu rootfs for RK3566 RK3568 RK3588 products](https://github.com/industrialtablet/Re-build-ubuntu20.04-rootfs-for-RK3566-RK3568-RK3588)



# Contacts

- Website: www.we-signage.com
- https://we-signage.en.made-in-china.com/
- E-mail: dennis@we-signage.com
- MP/Whatsapp/Wechat: + 86 13349909990
- Skype: solled686