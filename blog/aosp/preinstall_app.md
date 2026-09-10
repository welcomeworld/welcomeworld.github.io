# Android (内置)预装应用
参考[Android 系统如何预装第三方应用以及常见问题汇集](https://blog.csdn.net/csdn_pcb/article/details/85867426).
系统开机时会进行应用的加载(或者说安装)，所以我们大体上来说就是需要把要内置的apk放到指定目录就行。最常见的应该是下面四个
1.system/app/ ：该目录下存放的是一些系统级的应用，该目录下的应用能获取到比较高的权限，应用不可卸载，如Phone、Contacts等
2.system/priv-app/ ：该目录是从Android 4.4开始出现的目录，它存放的是一些系统核心应用，能获取到比system/app/下应用更高的权限，应用不可卸载，如:Setting、SystemUI等。
3.vendor/app/ ：该目录存放制造商的一些应用，应用不可卸载。
4.data/app/：该目录下存放的一些第三方应用，应用可卸载。用户手动安装的应用就是放在这个目录下
更多的可以看具体的代码，也可以自行添加。
源码是在PackageManagerService实例化时通过scanDirTracedLI方法进行扫描。
## 基础操作
首先，在源码目录下你喜欢的地方(一般应该是在packages/apps目录下新建子目录)新建一个Android.mk文件。输入以下内容
```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := Neteasemusic
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)
LOCAL_PROGUARD_ENABLED := disabled
```
然后，用你喜欢的方式向PRODUCT_PACKAGES变量追加你的项目名，比如我是在device/xiaomi/lavender/device.mk文件追加以下代码
```
PRODUCT_PACKAGES += Neteasemusic
```
## 差异化操作
#### 有源码(来自博客，未经验证)
将APK的Source code 拷贝至DemoApp下，删除/bin 和/gen目录
然后在文件最后追加下面几行代码
```
LOCAL_SRC_FILES := $(call all-subdir-java-files) 
LOCAL_CERTIFICATE := platform
include $(BUILD_PACKAGE)
```
#### 无源码
把apk文件复制到Android.mk同级目录，然后改个正常一点的名字(不知道会不会有影响，但是习惯性改个名字，不要有中文之类的非法字符)。然后在文件最后追加下面几行代码
```
LOCAL_SRC_FILES := 文件名.apk
#或者文件名改成与模块同名然后使用　LOCAL_SRC_FILES := $(LOCAL_MODULE).apk
LOCAL_MODULE_CLASS := APPS
LOCAL_CERTIFICATE := PRESIGNED
include $(BUILD_PREBUILT)
```
如果是通过BUILD_PREBUILT方式引入，编译系统会对文件签名有一定的改动，导致v2签名失效，v3大概也是不行的。所以要包含v1签名。如果保留原签名推荐使用下面完整示例里的shell方法直接copy文件过去，如果需要改成系统签名才使用BUILD_PREBUILT引入。
#### 非系统(可卸载)
在两个include之间追加下面代码
```
LOCAL_MODULE_PATH := $(TARGET_OUT_DATA_APPS)
```
然后修改位于frameworks/base/services/core/java/com/android/server/pm/的PackageManagerService文件，在第一次启动系统时扫描/data/app目录不传入SCAN_REQUIRE_KNOWN标志。
```
if(isFirstBoot()){
	scanDirTracedLI(sAppInstallDir, 0, scanFlags, 0);
}else {
	scanDirTracedLI(sAppInstallDir, 0, scanFlags|SCAN_REQUIRE_KNOWN, 0);
}
 ```
 因为Android9还是多少来着开始如果有这个标志会要求mSettings必须要有这个应用的PackageSetting存在(以前是如果存在会判断两者位置是否一致，后面追加了个else代码块，如果不存在也不行).很显然第一次启动的时候mSettings是没有东西的。
 

#### 系统(不可卸载)
在两个include之间追加下面代码
```
#如果是32位应用需要加这个标志
LOCAL_MULTILIB := 32
#在当前目录新建lib文件夹，将解压后的so库复制进去，然后一一引用(库多很麻烦，可以改用下面完整示例里的shell命令)
LOCAL_PREBUILT_JNI_LIBS := \
	lib/libijkffmpeg.so \
	lib/libijkplayer.so \
	lib/libijksdl.so
# 内置成核心应用，也就是内置到system/priv-app目录
LOCAL_PRIVILEGED_MODULE := true
```
系统应用的so库是到/system/lib或者/system/lib64目录下找的，所以需要单独将so库弄进相应目录，非系统应用不用。
## 完整代码示例
内置无源码网易云音乐为系统应用
```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := YZXNeteasemusic
LOCAL_MODULE_TAGS := optional
LOCAL_SRC_FILES := $(LOCAL_MODULE).apk
LOCAL_MODULE_CLASS := APPS
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)
LOCAL_CERTIFICATE := PRESIGNED
LOCAL_PROGUARD_ENABLED := disabled
include $(BUILD_PREBUILT)
#copy the nativelib to system/lib
$(shell cp $(LOCAL_PATH)/lib/armeabi/* $(TARGET_OUT)/lib/)
```
省事版
```
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := YZXNeteasemusic
# copy the apk to system/app/$(LOCAL_MODULE)
$(shell cp $(LOCAL_PATH)/$(LOCAL_MODULE).apk $(TARGET_OUT_APPS)/$(LOCAL_MODULE)/$(LOCAL_MODULE).apk)
#copy the nativelib to system/lib
$(shell cp $(LOCAL_PATH)/lib/* $(TARGET_OUT)/lib/)
```

## 代码说明
**LOCAL_PATH := $(call my-dir)**　固定开头，每个 Android.mk 文件必须以定义 LOCAL_PATH 为开始，它用于在开发 tree 中查找源文件

**include $(CLEAR_VARS)**　固定内容，紧跟LOCAL_PATH。CLEAR_VARS 变量由 Build System 提供，并指向一个指定的 GNU Makefile，由它负责清理除LOCAL_PATH以外的LOCAL_**变量

**LOCAL_MODULE_TAGS**：= user eng tests optional　表示在什么变体情况下编译该模块(apk)，一般不用写，默认是optional。这个变量已经基本废弃了。

**LOCAL_MODULE**　模块名，和其他模块不能重名，如果定义了LOCAL_PACKAGE_NAME就可以不用定义，因为默认赋值LOCAL_PACKAGE_NAME。

**LOCAL_CERTIFICATE**　表示apk的签名方式
testkey：普通 APK，默认情况下使用。
platform：该 APK 完成一些系统的核心功能。经过对系统中存在的文件夹的访问测试，
这种方式编译出来的 APK 所在进程的 UID 为 system，可以参见 Settings。
shared：该 APK 需要和 home/contacts 进程共享数据，可以参见 Launcher。
media：该 APK 是 media/download 系统中的一环，可以参见 Gallery。
PRESIGNED：使用apk原来的签名

**LOCAL_MODULE_CLASS**　指定模块的类型，可用于生成LOCAL_MODULE_PATH的默认值。可以不指定，因为BUILD_PACKAGE这些代码有指定，但是如果使用的是BUILD_PREBUILT代码，同时没有指定LOCAL_MODULE_PATH,那么应该是要指定的

**include $(BUILD_PACKAGE)**　表示生成一个 apk，它可以是多种类型
BUILD_PACKAGE（既可以编apk，也可以编资源包文件，但是需要指定LOCAL_EXPORT_PACKAGE_RESOURCES:=true)
BUILD_JAVA_LIBRARY（java共享库）
BUILD_STATIC_JAVA_LIBRARY（java静态库）
BUILD_EXECUTABLE（执行文件）
BUILD_SHARED_LIBRARY（native共享库）
BUILD_STATIC_LIBRARY（native静态库）

**LOCAL_PROGUARD_ENABLED** := disabled 指定不混淆代码，如果需要混淆可以通过LOCAL_PROGUARD_FLAGS 配置混淆规则

** LOCAL_MODULE_SUFFIX** 　模块名后缀(可选)，可以不指定

**LOCAL_PRIVILEGED_MODULE** := true表示apk将预装到system/priv-app/下

像**TARGET_OUT_APPS**这些变量在build/make/core/envsetup.mk里有所定义。

这个博客写的有点乱，因为情况比较多。如果有错漏希望指出。