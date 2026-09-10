# 重装Linux后系统配置（个人使用）
## 1、解决adb无法连接设备
在/etc/udev/rules.d目录下创建90-android.rules文件，输入
```
SUBSYSTEM=="usb", ENV{DEVTYPE}=="usb_device", MODE="0666"
```

## 2、vscode在makefile中的tab无法被make识别
在settings.json中添加这一项
```
    "editor.insertSpaces": false,
```
## 3. 修改gnome窗口标题栏大小
创建$HOME/.config/gtk-3.0/gtk.css文件,输入以下内容
```css
/*
 Decrease the size of head bars for non-CSD applications
 Gnome 20 (Fedora 24) compatible version
 https://unix.stackexchange.com/questions/276951/how-to-change-the-titlebar-height-in-standard-gtk-apps-and-those-with-headerbars
*/

/* x11 and xwayland windows */
window.ssd headerbar.titlebar {
    padding-top: 3px;
    padding-bottom: 3px;
    min-height: 0;
    /* remove border between titlebar and window */
    border: none;
    background-image: linear-gradient(to bottom,
     shade(@theme_bg_color, 1.05),
     shade(@theme_bg_color, 1.00));
    box-shadow: inset 0 1px shade(@theme_bg_color, 1.4);
}

window.ssd headerbar.titlebar button.titlebutton {
    padding: 0px;
    min-height: 0;
    min-width: 0;
}


/* native wayland ssd windows */
.default-decoration {
    padding: 3px;
    min-height: 0;
    /* remove border between titlebar and window */
    border: none;
    background-image: linear-gradient(to bottom,
     shade(@theme_bg_color, 1.05),
     shade(@theme_bg_color, 1.00));
    box-shadow: inset 0 1px shade(@theme_bg_color, 1.4);
}

.default-decoration .titlebutton {
    padding: 0px;
    min-height: 0;
    min-width: 0;
}
```

## 4.AndroidStudio 中Markdown无法预览
具体原因好像是Markdown插件换了一个新的渲染框架
Help->Find Action->Choose 然后选择带JCEF的JBR

## 5.AndroidStudio 无法输入中文
Help->Edit Custom VM 在后面加上
```
-Drecreate.x11.input.method=true
```
也是IDEA家族JBR的Bug，如果上面没有效果，貌似通过换修复过的JBR也可解决。

