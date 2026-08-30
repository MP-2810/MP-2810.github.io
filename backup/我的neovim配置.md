## Vim的一些基础配置
### 1. 行数
```
set number
set relativenumber
```
### 2. 缩进
```
set tabstop=4
set shiftwidth=4
set expandtab
set autoindent
set smartindent
```
## NeoVim的配置
### 1. 创建nvim的配置文件
```
mkdir -p ~/.config/nvim
cd ~/.config/nvim
touch init.lua
```

#### 拆分neovim配置
因为当配置文件庞大到一定程度之后不利于维护和阅读，因此可以拆分neovim配置来简化维护
通常的做法是在配置文件夹下创建一个lua文件夹，然后在其中添加各个模块：
例如有如下结构：
```
📁 nvim
-- 📄 init.lua
-- 📁 lua
---- 📄 module.lua
```
然后就可以在`init.lua`中使用`require("module")`引入`module.lua`

