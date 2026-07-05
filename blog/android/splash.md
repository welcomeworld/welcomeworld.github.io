# 牢骚（废话）
谷歌真是吃饱了撑的，搞这搞那，Android12新出了一个强制闪屏页，效果就跟小米的闪屏页广告一样，只不过谷歌的是强制的，小米是可选的。不过谷歌再怎么搞，你还是要适配啊，难受。
# 正题
所有运行在Android12上的应用（无论targetSdkVersion是多少）在冷启动(应用进程未运行)和温启动（应用进程在后台但是没有Activity,比如存在Service）时都会强制强制地弹SplashScreen。官方的API是没有办法禁用SplashScreen的。闪屏固定样式如下。
![在这里插入图片描述](../../static/img/blog_splash_style.png#pic_center)
# 适配
## 12以下照常，12以上先系统后应用闪屏
如果可以接受弹完系统闪屏再弹应用的闪屏，那什么都不需要做，默认就这个效果。
## 12以下照常，12以上只系统闪屏
首先compileSdkVersion要设置在31以上，不然没有12的api。接着在SplashActivity的onCreate方法加上如下代码

```java
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
            getSplashScreen().setOnExitAnimationListener(view -> {
                //do nothing to keep splash screen
            });
        }
```
原理是如果我们不注册这个监听，系统闪屏页在应用开始绘制后就自行消失。但如果我们注册了这个监听，系统在应用开始绘制之后会将系统闪屏页以SplashScreenView的形式回调给应用，由开发者决定系统闪屏页怎么消失。比如我们可以自定义动画上移下移或者渐隐等方式让系统闪屏页消失。我们注册了OnExitAnimationListener，什么都不做系统闪屏页就会一直覆盖在闪屏页上，直到应用跳转到主Activity。或者可以缓存SplashScreenView等广告什么的加载好了再remove掉隐藏它。
## 所有系统版本统一12样式
这个可以使用官方的SplashScreen compat library，下面的链接有官方文档，不过目前还是beta版本，这个自行衡量。如果自己实现的话就是将以前的闪屏图片改成12的样式，这个应该不难，12的样式挺简单的，然后根据上面的说明在12以上屏蔽掉应用的闪屏就好了。
# 系统闪屏样式配置
12的闪屏流程可以分为两步。
一个是进场，也就是点击应用后立刻弹出的上面固定样式,通过主题配置。
可以设置闪屏背景，不过只能是颜色背景，无法设置图片。

```xml
<item name="windowSplashScreenBackground">#CCCCCC</item>
```
可以设置中间的闪屏图标，这个可以设置图片，不过显示的大小和样式（圆的方的）由系统决定,会通过Mask遮蔽实现。

```xml
<item name="android:windowSplashScreenAnimatedIcon"></item>
```
可以设置商标图案，根据郭霖大神的测试应该是4：1的比例图片，否则会被拉伸.

```xml
<item name="android:windowSplashScreenBrandingImage"></item>
```
系统闪屏能设置的就这三个。
另一个就是退场，也就是应用开始绘制后的退场处理，如果没有适配系统闪屏将直接消失，如果注册了上面的监听则用户自行处理，这时候就是闪屏页就是普通View,应用可以完全操控。

# 参考文章
[Migrate your existing splash screen implementation to Android 12 and higher ](https://developer.android.com/guide/topics/ui/splash-screen/migrate)  
[Android 12 SplashScreen API快速入门](https://blog.csdn.net/guolin_blog/article/details/120275319)  
[深度探讨如何使用 Jetpack SplashScreen 重塑应用启动画面](https://baijiahao.baidu.com/s?id=1716545016612766310&wfr=spider&for=pc)

