## linuxqq无法正常使用中文输入法的问题
正常安装了fcitx5，并且在其他地方如终端、浏览器等地方都可以正常使用，但qq却出现了无法切换输入法、设置为中文输入法也无法输出中文拼音拼写的情况
- 此前先安装过KDE,在KDE中qq可以正常使用中文输入法

可能是会话环境或 XWayland 输入桥接有问题，如果没特殊情况的话，输入命令`hyprctl clients`，应该能查看到qq是依赖xwayland运行的
通过编辑`hyprland.lua`配置文件可以解决：
添加环境设置：
```
hl.env("GTK_IM_MODULE", "fcitx")
-- hl.env("QT_IM_MODULE", "fcitx")
hl.env("XMODIFIERS", "@im=fcitx")
```