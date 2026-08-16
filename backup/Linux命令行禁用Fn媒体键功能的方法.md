1. 查看当前模式
`cat /sys/module/hid_apple/parameters/fnmode`
    0：禁用媒体功能，F键直接触发标准功能（如F1全屏）
    1：媒体优先（默认）
    2：F键优先（标准F功能，Fn+F1~F12触发媒体功能）
2. 临时检验
`echo 2 | sudo tee /sys/module/hid_apple/parameters/fnmode`
3. 创建模块配置文件
`echo "options hid_apple fnmode=2" | sudo tee /etc/modprobe.d/hid_apple.conf`
4. 更新 initramfs
`sudo mkinitcpio -P`

- 如果外接键盘不识别 hid_apple 模块，可以试试另一个内核参数：
- `echo 2 | sudo tee /sys/module/hid_logitech_hidpp/parameters/disable_raw_mode`