@[TOC](Android签名修改)
# 参考文章
[简书链接](https://www.jianshu.com/p/bb5325760506)
# 签名文件存放目录
/build/target/product/security

# 参考文件
/build/target/product/security/README

# 操作流程
## 替换文件
### 示例代码
```
./development/tools/make_key build/target/product/security/shared '/C=CN/ST=ShenZhen/L=ShenZhen/O=momxmo/OU=mo/CN=www.momxmo.com/emailAddress=test@126.com'
```
注意，不要设置密码，直接回车
### verity_key文件需要多一步生成
先生成 generate_verity_key
```
make generate_verity_key
```
大概是等效
```
 mmm system/extras/verity/
```
然后
```
out/host/linux-x86/bin/generate_verity_key -convert veritykey.x509.pem verity_key
```
最后拷贝veritykey.pk8，veritykey.x509.pem，verity_key.pub 至 build/target/product/security/ 目录，将其重命名: verity.pk8， verity.x509.pem，verity_key 。
## 修改配置
1.修改/build/core/config.mk文件
```
DEFAULT_SYSTEM_DEV_CERTIFICATE := build/target/product/security/releasekey
```
2.修改/build/core/Makefile.mk文件
```
ifeq ($(DEFAULT_SYSTEM_DEV_CERTIFICATE),build/target/product/security/releasekey)  
    BUILD_VERSION_TAGS += release-keys 
```
3.修改system/sepolicy/private/keys.conf 和 system/sepolicy/prebuilts/api/{apilevel}/private/keys.conf文件
```
[@RELEASE]
ENG       : $DEFAULT_SYSTEM_DEV_CERTIFICATE/releasekey.x509.pem
USER      : $DEFAULT_SYSTEM_DEV_CERTIFICATE/releasekey.x509.pem
USERDEBUG : $DEFAULT_SYSTEM_DEV_CERTIFICATE/releasekey.x509.pem
```