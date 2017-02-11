
# 环境
  ubuntu 16.04LTS x86_64<br>
  android-ndk64-r19-linux-x86_64<br>
  ffmpeg 3.0.5<br>

# 使用
##Step 1
安装android linux NDK以及SDK，并配置环境变量；<br>
从[ffmpeg官网](http://ffmpeg.org/)下载ffmpeg源码包。<br>

##Step 2
修改ffmpeg/configure文<br>
将
```
SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR)$(SLIBNAME)'
```
修改为：<br>
```
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'
```
这样编译出来的so命名才符合android的使用。
##Step 3
本项目提供了分别编译arm平台库和x86库的sh文件，分别为`source/build_android_arm.sh` 以及 `source/build_android_x86.sh`<br>
下面以build_android_arm.sh为例进行说明：<br>
将source/build_android_arm.sh复制到ffmpeg目录下，并修改build_android_arm.sh中的 TMPDIR、NDK、SYSROOT、TOOLCHAIN、PREFIX变量为自己的具体情况，具体如下：<br>
#####1.指定临时目录
```
export TMPDIR=$HOME/temp
```
指定一个临时目录，可以是任何路径，但必须保证存在，ffmpeg编译要用；<br>
#####2.指定NDK路径
```
NDK=$HOME/Android/Sdk/ndk-bundle
```
#####3.指定使用NDK Platform版本
```
SYSROOT=$NDK/platforms/android-19/arch-arm/
```
这里指定的ndk platform的路径，一定要`选择比你的目标机器使用的版本低的`，比如你的手机是android-15版本，那么就选择低于15的<br>
#####4.指定编译工具链
```
TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64
```
#####5.指定编译后的安装目录
```
PREFIX=$HOME/ffmpeg-android/arm/
```
这个目录是ffmpeg编译后的so的输出目录，会有一个include和lib文件夹生成在这里，这也是我们之后要在android apk中使用的.<br>
<br>
#####build_android_arm.sh示例
可以修改该文件来控制ffmpeg的编译config来达到自己想要的库文件，我这里为了得到动态链接库，--enable-shared，并--disable-static，我开放了所有的编解码器，如果有不需要的，可以通过--disable-coder和--disable-decoder来指定，具体查看ffmpeg文档.


```bash
#!/bin/bash
export TMPDIR=$HOME/temp
NDK=$HOME/Android/Sdk/ndk-bundle
SYSROOT=$NDK/platforms/android-19/arch-arm/
TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64

CPU=arm
PREFIX=$HOME/ffmpeg-android/arm/
ADDI_CFLAGS="-marm"

function build_one
{
./configure \
--prefix=$PREFIX \
--enable-shared \
--disable-static \
--disable-doc \
--disable-ffmpeg \
--disable-ffplay \
--disable-ffprobe \
--disable-ffserver \
--disable-doc \
--disable-symver \
--enable-small \
--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
--target-os=linux \
--arch=arm \
--enable-cross-compile \
--sysroot=$SYSROOT \
--extra-cflags="-Os -fpic $ADDI_CFLAGS" \
--extra-ldflags="$ADDI_LDFLAGS" \
$ADDITIONAL_CONFIGURE_FLAG
make clean
make
make install
}

build_one
```
##Step 4
```
将build_android_arm.sh复制到ffmpeg目录下
来到ffmpeg目录
./build_andrioid_arm.sh
```

##Step 5
等待一段时间后，会在 $PREFIX 目录下生成 include和lib两个文件夹，将lib文件夹中的 pkgconfig 目录和so的链接文件删除，只保留so文件，然后将include 和lib两个目录一起copy到你的apk jni下去编译，具体请参考我的 blog [在 Android Studio 2.2 中愉快地使用 C/C++](https://my.oschina.net/u/735973/blog/812920)

# Reference & Thanks
  [android-ffmpeg-tutorial](https://github.com/roman10/android-ffmpeg-tutorial)
