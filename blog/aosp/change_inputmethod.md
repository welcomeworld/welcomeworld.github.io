# Android系统修改默认输入法
代码是lineageos17.1(lavender)
## 一、内置输入法进系统
[Android (内置)预装应用](https://blog.csdn.net/Welcome_Word/article/details/114435483).
## 二、修改内置输入法
在frameworks/base/packages/SettingsProvider/res/values/defaults.xml文件后面追加下面代码
```
     <string name="def_input_method" translatable="false">com.iflytek.inputmethod/.FlyIME</string>
```
上面这个字符串视要内置的输入法而定，这个是讯飞的，当然也可以不要这段代码，下面直接硬编码。
然后在frameworks/base/packages/SettingsProvider/src/com/android/providers/settings/DatabaseHelper.java文件的loadSecureSettings(SQLiteDatabase db)方法后面追加下面代码
```
loadStringSetting(stmt, Settings.Secure.DEFAULT_INPUT_METHOD,R.string.def_input_method);
loadStringSetting(stmt, Settings.Secure.ENABLED_INPUT_METHODS,R.string.def_input_method);
```
就行了。
## 三、切换语言问题
切换语言的时候调用frameworks/base/services/core/java/com/android/server/inputmethod/InputMethodUtils.java这个文件的isSystemImeThatHasSubtypeOf方法判断输入法是否支持这个语言，在这个方法里面添加下面代码直接返回true就行了。
```
        if ("com.iflytek.inputmethod".equals(imi.getPackageName())) {
            return true;
        }
```
## 四、附加信息
默认输入法源码在packages/inputmethods目录下