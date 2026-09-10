# 裁减编译生成目标减少编译时间
## 跳过生成OTA包
参考链接[简书博客](https://www.jianshu.com/p/1b3ff36f4138).
详细的分析上面的博客说的很清楚了
具体操作就是在所选用的device中BoardConfig.mk文件，修改或者增加一行TARGET_SKIP_OTA_PACKAGE := true 即可在构建时不生成ota更新包。
## 跳过生成userdata.img
在build/make/core/main.mk文件里定义了userdataimage的伪目标，代码如下
```
.PHONY: userdataimage
userdataimage: $(INSTALLED_USERDATAIMAGE_TARGET)
```

INSTALLED_USERDATAIMAGE_TARGET在build/make/core/Makefile文件里定义,定位代码
```
ifdef BUILDING_USERDATA_IMAGE
userdataimage_intermediates := \
    $(call intermediates-dir-for,PACKAGING,userdata)
BUILT_USERDATAIMAGE_TARGET := $(PRODUCT_OUT)/userdata.img

define build-userdataimage-target
  $(call pretty,"Target userdata fs image: $(INSTALLED_USERDATAIMAGE_TARGET)")
  @mkdir -p $(TARGET_OUT_DATA)
  @mkdir -p $(userdataimage_intermediates) && rm -rf $(userdataimage_intermediates)/userdata_image_info.txt
  $(call generate-image-prop-dictionary, $(userdataimage_intermediates)/userdata_image_info.txt,userdata,skip_fsck=true)
  $(hide) PATH=$(foreach p,$(INTERNAL_USERIMAGES_BINARY_PATHS),$(p):)$$PATH \
      build/make/tools/releasetools/build_image.py \
      $(TARGET_OUT_DATA) $(userdataimage_intermediates)/userdata_image_info.txt $(INSTALLED_USERDATAIMAGE_TARGET) $(TARGET_OUT)
  $(hide) $(call assert-max-image-size,$(INSTALLED_USERDATAIMAGE_TARGET),$(BOARD_USERDATAIMAGE_PARTITION_SIZE))
endef

# We just build this directly to the install location.
INSTALLED_USERDATAIMAGE_TARGET := $(BUILT_USERDATAIMAGE_TARGET)
INSTALLED_USERDATAIMAGE_TARGET_DEPS := \
    $(INTERNAL_USERIMAGES_DEPS) \
    $(INTERNAL_USERDATAIMAGE_FILES) \
    $(BUILD_IMAGE_SRCS)
$(INSTALLED_USERDATAIMAGE_TARGET): $(INSTALLED_USERDATAIMAGE_TARGET_DEPS)
	$(build-userdataimage-target)

.PHONY: userdataimage-nodeps
userdataimage-nodeps: | $(INTERNAL_USERIMAGES_DEPS)
	$(build-userdataimage-target)

endif # BUILDING_USERDATA_IMAGE
```

而BUILDING_USERDATA_IMAGE在build/make/core/board_config.mk文件里定义
```
# Are we building a userdata image
BUILDING_USERDATA_IMAGE :=
ifeq ($(PRODUCT_BUILD_USERDATA_IMAGE),)
  ifdef BOARD_USERDATAIMAGE_PARTITION_SIZE
    BUILDING_USERDATA_IMAGE := true
  endif
else ifeq ($(PRODUCT_BUILD_USERDATA_IMAGE),true)
  BUILDING_USERDATA_IMAGE := true
endif
.KATI_READONLY := BUILDING_USERDATA_IMAGE
```
到这里就很明确了，也就是在编译目标的BoardConfig.mk设置
```
PRODUCT_BUILD_USERDATA_IMAGE := false
```
或者一般来说不设置BOARD_USERDATAIMAGE_PARTITION_SIZE也行