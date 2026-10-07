### 由AI负责整理
# Linux QQ 中文输入异常

## 1. 现象
在 Arch Linux + Hyprland（Wayland）环境下使用 Fcitx5 中文输入法时：

* Firefox、Kitty 等程序输入正常；
* Linux QQ 输入中文时，偶尔会有拼音字母直接进入输入框；
* 输入速度越快，出现概率越高；
* Fcitx5 候选框在 QQ 中会出现闪烁或输入状态异常

因此问题只存在于 Linux QQ，并非 Fcitx5 的全局配置问题

## 2. 原因
Linux QQ 当前使用 Wayland IME：

```text
--enable-wayland-ime
--wayland-text-input-version=3
```

其 Wayland 输入法处理方式与 Fcitx5 存在兼容性问题，导致 QQ 在接收 Fcitx5 输入事件时出现异常

通过为 QQ 单独设置：
```text
QT_IM_MODULE=fcitx
GTK_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```

可以让 QQ 使用兼容 Fcitx5 的输入法路径，从而解决中文输入异常

由于 Firefox、Kitty 等程序本身输入正常，因此不应该将这些变量设置为全局环境变量，而应该只针对 QQ 设置

## 3. 解决方法

### 3.1 确认 QQ 的启动参数

查看 QQ 进程：
```bash
ps -ef | grep '[q]q'
```

可以看到类似：
```text
/opt/QQ/qq --enable-wayland-ime --wayland-text-input-version=3
```

说明 QQ 启用了 Wayland IME。

### 3.2 确认 Desktop Entry

搜索 QQ 的 `.desktop` 文件：
```bash
grep -Ril "linuxqq\|/opt/QQ/qq" \
    /usr/share/applications \
    ~/.local/share/applications 2>/dev/null
```

使用用户目录中的：
```text
~/.local/share/applications/qq.desktop
```

不要直接修改：
```text
/usr/share/applications/qq.desktop
```

这样可以避免软件更新覆盖自己的配置

### 3.3 修改 QQ 的启动命令

编辑：
```bash
nvim ~/.local/share/applications/qq.desktop
```

将原来的`exec`一行：
```ini
Exec=linuxqq %U --enable-wayland-ime --wayland-text-input-version=3
```

修改为：
```ini
Exec=env QT_IM_MODULE=fcitx GTK_IM_MODULE=fcitx XMODIFIERS=@im=fcitx linuxqq %U
```

我的完整配置如下：
```ini
[Desktop Entry]
Name=QQ
Exec=env QT_IM_MODULE=fcitx GTK_IM_MODULE=fcitx XMODIFIERS=@im=fcitx linuxqq %U
Terminal=false
Type=Application
Icon=qq
StartupWMClass=QQ
Categories=Network;
Comment=QQ
```

### 3.4 重启 QQ

完全退出 QQ：

```bash
pkill qq
```

然后重新通过 Fuzzel 或其他应用启动器启动 QQ

之后 Linux QQ 会单独使用：

```text
QT_IM_MODULE=fcitx
GTK_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```

而 Firefox、Kitty 等其他程序不会受到影响

## 4. 最终结果

最终配置关系：

```text
Fcitx5
 │
 ├── Firefox → 原有 Wayland 输入方式
 │
 ├── Kitty   → 原有输入方式
 │
 └── Linux QQ
       │
       └── QT_IM_MODULE=fcitx
           GTK_IM_MODULE=fcitx
           XMODIFIERS=@im=fcitx
```

Linux QQ 中文输入恢复正常，同时不会改变系统其他程序的输入法配置。

**核心解决方案：不修改全局 Fcitx5 环境，只通过用户级 `qq.desktop` 为 Linux QQ 单独指定 Fcitx5 输入环境。**
