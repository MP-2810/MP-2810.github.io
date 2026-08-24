### 在使用过程中发现，有出现终端显示正常，但输入法、浏览器各种无法正常显示各种小图标的问题，如输入法不能显示，但是浏览器却能显示。一般是没能正常安装emoji字体以及只给终端设置了使用字体导致的割裂

## 确保安装emoji字体
`sudo pacman -S noto-fonts-emoji`
## 然后设置字体回退顺序
```
mkdir -p ~/.config/fontconfig
vim ~/.config/fontconfig/fonts.conf
```
添加：
```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>

    <match>
        <test name="family">
            <string>sans-serif</string>
        </test>
        <edit name="family" mode="append">
            <string>Noto Color Emoji</string>
        </edit>
    </match>

    <match>
        <test name="family">
            <string>monospace</string>
        </test>
        <edit name="family" mode="append">
            <string>Noto Color Emoji</string>
        </edit>
    </match>

</fontconfig>
```
然后刷新缓存
`fc-cache -fv`
最后完全关闭浏览器后再打开检查