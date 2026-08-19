## **窗口透明度**
`hyprland.lua`中搜索`gaps_in`找到对应的配置模块，其中参数可以调节窗口圆角、透明度和模糊指数等参数
不建议设置全局透明度，因为会导致正常画面也会被透明化例如浏览器放视频时的视频画面
搜索`window_rule`来到文件最底层，可以新添窗口规则设置指定窗口透明化
例子：这里新添一条名为`kitty-opacity`的指定窗口透明化规则
```lua
hl.window_rule({
    name = "kitty-opacity",
    match = { class = "kitty"}，
    opacity = "0.8 override 0.82 override 0.8", -- 意思是聚焦窗口透明度0.8、非聚焦窗口透明度0.82和全屏时透明度0.8
-- 也可以直接简写成"0.8 0.82 0.8"
}) 
```

## **全屏快捷键**
hyprland没有默认的全屏快捷键设置方案，可以手动添加，但是全屏调度器使用稍微有点复杂：
```lua
hl.bind(mainMod .. "F", hl.dsp.window.fullscreen_state({
    internal = 2,   -- 1=最大化，2=全屏
    client = 1,     -- 1=当前窗口，2=所有窗口
    action = "toggle" -- "toggle" 切换全屏状态，"on" 强制全屏，"off" 强制退出全屏
}))
```

## **greetd**
安装greetd:
`sudo pacman -S greetd`
编辑配置文件：
`sudo vim /etc/greetd/config/config.toml`
将`command`参数中`agreety --cmd /bin/sh`的`sh`换成`start-hyprland`即可实现进入账户时hyprland的自动启动
- 更高级的登录美化功能暂待发掘