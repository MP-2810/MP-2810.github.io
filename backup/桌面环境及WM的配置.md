**配置KDE plasma和Hyprland**

## 配置KDE plasma
### 安装kde相关软件包、中文字体、输入法及firefox
`sudo pacman -S plasma-meta kde-utilities-meta kde-system-meta noto-fonts-cjk fcitx5 fcitx5-gtk fcitx5-chinese-addons firefox firefox-i18n-zh-cn libreoffice-still libreoffice-still-zh-cn`
**安装选择：**
个人使用`ffmpeg`、`pipewire-jack`(因为已经安装了pipewire相关包(上篇))、`noto-fonts`、`16`（简体中文）、`cronie`
**配置环境文件：**
sddm不会读取每一个用户自己的locale配置，因为系统全局设置美式英文避免终端中出现豆腐块文字
`vim /usr/lib/systemd/system/sddm.service`
在`[Service]`项下面添加`Environment=LANG=zh_CN.UTF-8`以设置sddm本地化为中文
- 倘若遇到了sddm.service文件不存在的问题，可以重新安装sddm修复：
- `pacman -S sddm --overwrite='/usr/lib/systemd/system/sddm.service'`
- - 可以考虑额外安装kdesu来为使用root命令提供便捷的GUI服务
### 启用sddm服务
`systemctl enable --now sddm.service`
进入kde后，可以修改主题，包括进入kde时的用户登录界面主题，但修改后者需要root权限，别忘了给自己的个人用户添加进wheel组提供获取root的权限
- 可能会遇到设置里没有直接设置sddm界面主题的选项，可以使用命令行手动修改：
- 先查看有哪些可用主题：
- `ls /usr/share/sddm/themes`
- 然后修改sddm配置文件：
- `sudo vim /etc/sddm.comf`
- 写入：
```
[Theme]
Current=主题名字
```
- 重启sddm生效：
- `sudo systemctl restart sddm`

### 一些基础设置和软件
1. kate是标配的文本编辑器
2. konsole是标配的文档管理器
3. ark是标配的压缩解压缩软件
4. haruna是基于Qt和MPV的视频播放器
5. gwenview是标配的图片查看工具
- 如果没有可以手动安装一下
6. 进入设置，将虚拟键盘设置成`Fcitx 5 Wayland启动器`
7. 可以在无障碍服务里的抖动指针选项关闭抖动时放大鼠标指针的功能
8. 右键右下角的时间组件，选择“配置 数字时钟”可以将中国农历添加进日历界面

## 配置Hyprland
