# 音视频系列一 Linux 编译ffmpeg4.2.4
[参考文章](https://juejin.cn/post/6844903945496690696)
## 一 下载解压ndk
使用的是ndk21(编译器是clang的应该都一样，低版本gcc的配置会有不同)
## 二 下载解压ffmpeg
[ffmpeg下载地址](http://ffmpeg.org/releases)
选4.2.4主要考虑既不太新又不太旧
## 三 编写编译脚本
```bash
#!/bin/bash

NDK=/PublicData/LinuxAndroidSdk/ndk/21.3.6528147
TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/linux-x86_64
SYSROOT="$TOOLCHAIN/sysroot"
API=21

function build_android
{
echo "Compiling FFmpeg for $CPU"
make clean
./configure \
    --prefix=$PREFIX \
    --libdir=$LIB_DIR \
    --enable-shared \
    --disable-static \
    --enable-jni \
	--enable-gpl \
    --disable-doc \
	--disable-ffmpeg \
	--disable-ffplay \
	--disable-ffprobe \
    --disable-symver \
    --disable-programs \
    --target-os=android \
    --arch=$ARCH \
    --cpu=$CPU \
    --cc=$CC \
    --cxx=$CXX \
    --enable-cross-compile \
	--cross-prefix=$CROSS_PREFIX \
    --sysroot=$SYSROOT \
    --extra-cflags="-Os -fpic $OPTIMIZE_CFLAGS" \
    --extra-ldflags="$ADDI_LDFLAGS" \
    --disable-asm \
    --disable-yasm \
    $ADDITIONAL_CONFIGURE_FLAG
make clean
make -j10
make install
echo "The Compilation of FFmpeg for $CPU is completed"
}

# armv8-a
OUTPUT_FOLDER="arm64-v8a"
ARCH=arm64
CPU="armv8-a"
TOOL_CPU_NAME=aarch64
TOOL_PREFIX="$TOOLCHAIN/bin/$TOOL_CPU_NAME-linux-android"

CC="$TOOL_PREFIX$API-clang"
CXX="$TOOL_PREFIX$API-clang++"
PREFIX="${PWD}/android/$OUTPUT_FOLDER"
CROSS_PREFIX=$TOOLCHAIN/bin/aarch64-linux-android-
LIB_DIR="${PWD}/android/libs/$OUTPUT_FOLDER"
OPTIMIZE_CFLAGS="-march=$CPU"
build_android

# # armv7-a
# OUTPUT_FOLDER="armeabi-v7a"
# ARCH="arm"
# CPU="armv7-a"
# TOOL_CPU_NAME=armv7a
# TOOL_PREFIX="$TOOLCHAIN/bin/arm-linux-androideabi"

# CC="$TOOLCHAIN/bin/armv7a-linux-androideabi$API-clang"
# CXX="$TOOLCHAIN/bin/armv7a-linux-androideabi$API-clang++"
# PREFIX="${PWD}/android/$OUTPUT_FOLDER"
# CROSS_PREFIX=$TOOLCHAIN/bin/arm-linux-androideabi-
# LIB_DIR="${PWD}/android/libs/$OUTPUT_FOLDER"
# OPTIMIZE_CFLAGS="-march=$CPU"
# build_android

# # x86
# OUTPUT_FOLDER="x86"
# ARCH="x86"
# CPU="x86"
# TOOL_CPU_NAME="i686"
# TOOL_PREFIX="$TOOLCHAIN/bin/${TOOL_CPU_NAME}-linux-android"

# CC="$TOOL_PREFIX$API-clang"
# CXX="$TOOL_PREFIX$API-clang++"
# PREFIX="${PWD}/android/$OUTPUT_FOLDER"
# CROSS_PREFIX=$TOOLCHAIN/bin/i686-linux-android-
# LIB_DIR="${PWD}/android/libs/$OUTPUT_FOLDER"
# OPTIMIZE_CFLAGS="-march=i686 -mtune=intel -mssse3 -mfpmath=sse -m32"
# build_android

# # x86_64
# OUTPUT_FOLDER="x86_64"
# ARCH="x86_64"
# CPU="x86-64"
# TOOL_CPU_NAME="x86_64"
# TOOL_PREFIX="$TOOLCHAIN/bin/${TOOL_CPU_NAME}-linux-android"

# CC="$TOOL_PREFIX$API-clang"
# CXX="$TOOL_PREFIX$API-clang++"
# PREFIX="${PWD}/android/$OUTPUT_FOLDER"
# CROSS_PREFIX=$TOOLCHAIN/bin/x86_64-linux-android-
# LIB_DIR="${PWD}/android/libs/$OUTPUT_FOLDER"
# OPTIMIZE_CFLAGS="-march=$CPU"
# build_android
```
**NDK**修改成自己的目录，设置**target-os=android**可以不用修改./configure文件，详细的disable和enable配置通过
```bash
./configure --help
```
查看
## 四 编译
文件授权
```bash
chmod +x ./build_android.sh
```
运行
```bash
./build_android.sh
```
等待编译完成，没有任何问题的话就可以在./android的子目录下看到那些so文件