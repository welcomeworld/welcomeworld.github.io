# Android编译ota包移除compatibility.zip
## 简介
Android有个Treble项目，详细的情况感兴趣的自行百度，简单的来说就是编译一个系统镜像能够在很多手机上运行。也就涉及到了兼容性问题，compatibility.zip里面有四个文件，分别说明，设备提供了什么，设备需要什么，框架提供了什么，框架需要什么。如果ota包，也就是卡刷包里面有这个压缩包，那么recovery会去校验，如果不匹配的话无法安装。相关链接:[谷歌文档](https://source.android.google.cn/devices/architecture/vintf).

## 操作
顺着make otapackage 这个命令找，最后定位到了build/make/tools/releasetools/ota_from_target_files.py这个文件,里面有个AddCompatibilityArchiveIfTrebleEnabled方法，顾名思义。往下看有一处代码

```
  if not HasTrebleEnabled(target_zip, target_info):
    return
```
然后

```
def HasTrebleEnabled(target_files_zip, target_info):
  return (HasVendorPartition(target_files_zip) and
          target_info.GetBuildProp("ro.treble.enabled") == "true")
```
将HasTrebleEnabled方法改成返回False就行了