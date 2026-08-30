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
# NeoVim的配置
## 1. 创建nvim的配置文件
```
mkdir -p ~/.config/nvim
cd ~/.config/nvim
touch init.lua
```

### 拆分neovim配置
因为当配置文件庞大到一定程度之后不利于维护和阅读，因此可以拆分neovim配置来简化维护
通常的做法是在配置文件夹下创建一个lua文件夹，然后在其中添加各个模块：
例如有如下结构：
```
📁 nvim
-- 📄 init.lua
-- 📁 lua
---- 📄 module.lua
---- 📁 core
------ 📄 module.lua
```
然后就可以在`init.lua`中使用`require("module")`引入`module.lua`
使用`require("core.module")`引入core/module.lua

## 2. 一些基础配置
### 行号和相对行号
```
vim.opt.number = true
vim.opt.relativenumber = true
```
### 行列高亮显示
```
vim.opt.cursorline = true
vim.opt.colorcolumn = "100"
```
### 缩进
```bash
vim.opt.expandtab = true  #按下tab自动转换为空格
vim.opt.tabstop = 4  #一个制表符对应的空格数
vim.opt.shiftwidth = 0  #新建行时按tab会读取shiftwidth并写入对应的空格数
```
### 自动加载外部修改
`vim.opt.autoread = true`