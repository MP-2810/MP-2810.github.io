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

## 3. 快捷键
撤销ctrl+z和取消撤销的快捷键
```
vim.keymap.set({ "n", "i" }, "<C-z>", "<Cmd>undo<CR>", { silent = true })
vim.keymap.set( "i", "<C-r>", "<Cmd>redo<CR>", {silent = true })

vim.g.mapleader = " "
```

## 4. 插件
### 安装lazy插件管理器
#### 先安装git
`sudo pacman -S git`
#### 然后编写配置文件
例如在lua/core里添加lazy.lua文件
写入
```
local lazypath = vim.fn.stdpath("data") .. "lazy/lazy.nvim"

if not vim.uv.fs_stat(lazypath) then
    vim.fn.system({
        "git",
        "clone",
        "--filter=blob:none",
        "https://github.com/folke/lazy.nvim.git",
        "--branch=stable",
        lazypath
    })
end

vim.opt.rtp:prepend(lazypath)

require("lazy").setup({})
```
以设置lazy的自动安装及其检测
然后重启nvim,等待安装完毕
输入`:Lazy`看是否能打开lazy面板检验安装

#### 先安装个主题看看
在lazy.lua中改写如下内容
```
require("lazy").setup({
    spec = {
        { import = "plugins" }
    }
})
```
然后创建与core同级的目录`plugins`
创建tokyonight.lua
写入
```
return {
    "folke/tokyonight.nvim"
}
```
然后重启nvim,等待lazy自动clone
如果出现无法克隆的情况，查看我的博客`配置SSH使用`配置SSH
完成后输入`:colorscheme tokyonight`即可启用主题
可以编辑tokyonight.lua来实现自动启用主题
改写为
```
return {
    "folke/tokyonight.nvim",
    opts = {
        style = "moon"
    },
    config = function (_, opts)
        require("tokyonight").setup(opts)
        vim.cmd("colorscheme tokyonight")
    end
}
```

## 5. 优化编辑体验
### 自动补全括号/引号
在plugins下新建nvim-autopairs.lua，写入
```
return {
    "windwp/nvim-autopairs",
    event = "InsertEnter",
    opts = {}
}
```

### 添加括号/引号/标签
在plugins下新建nvim-surround.lua，写入
```
return {
    "kylechui/nvim-surround",
    event = "VeryLazy",
    opts = {}
}
```
比如`ysiw "`可以将目标单词用引号包裹
`ys2w "`可以将光标移动范围（2w）内用引号包裹
也可以按`v`进入可视模式来选中范围，按`S 引号`来包裹
`ds "`可以删除光标内容外的一对引号
`cs " '`可以把双引号改成单引号

### 快速跳转
在plugins下新建hop.lua，写入
```
return {
    "smoka7/hop.nvim",
    opts = {
        hint_position = 3,
    },
    keys = {
        { "<leader>hp", ":HopWord<CR>", silent = true }
    }
}
```

## 6. Buffer/Window/Tab
✔buffer：文件在内存当中的表示
✔window：显示buffer的视窗，一个窗口同一时间存在一个buffer
✔tab page：window的集合，一个neovim session可以有多个window
在plugins下新建bufferline.lua，写入
```
return {
    "akinsho/bufferline.nvim",
    dependencies = {
        "nvim-tree/nvim-web-devicons"
    },
    opts = {},
    keys = {
        { "<leader>bb", ":BufferLineCyclePrev<CR>", silent = true },
        { "<leader>bn", ":BufferLineCycleNext<CR>", silent = true },
        { "<leader>bp", ":BufferLinePick<CR>", silent = true },
        { "<leader>bd", ":bdelete<CR>", silent = true},
    },
    lazy = false
}
```
###  懒加载插件
在插件数多时提前加载全部插件会影响启动速度
因此懒加载提供了只在某些契机下才启用某插件的功能
✔event：在某个事件触发的时候加载插件
✔cmd：在某个命令被执行的时候加载插件
✔ft：当前buffer为特定文件类型的时候加载插件
✔keys：当触发快捷键时加载插件，如果快捷键不存在则创建快捷键
因此bufferline.lua中有一行`lazy = false`用于禁用懒加载，使其可以在nvim启动时就启用bufferline插件

## 7. LSP
使用Mason安装语言服务器
依赖以下组件
✔git
✔curl
✔unzip
✔tar
✔gzip
### 基本配置（以lua为例）
在plugins下新建mason.lua，写入
```
return {
    "williamboman/mason.nvim",
    event = "VeryLazy",
    opts = {}
}
```
mason提供了`Mason`命令打开面板
执行`:Mason`，会自动下载可安装工具列表的索引文件
如果出现报错无法下载，执行
`cd ~/.local/share/nvim/mason/registries/github/mason-org/mason-registry/`
检查是否有 registry.json 文件，如果没有，说明下载完全不成功，我们可以手动克隆注册表：
`git clone https://github.com/mason-org/mason-registry.git .`
如果只有 registry.json，可以手动压缩它：
`zip registry.json.zip registry.json`
然后重启nvim，执行`:Mason`
然后再在mason.lua里继续写入
```
return {
    "mason-org/mason.nvim",
    event = "VeryLazy",
    dependencies = {
        "neovim/nvim-lspconfig",
        "mason-org/mason-lspconfig.nvim",
    },
    opts = {},
    config = function (_, opts)
        require("mason").setup(opts)

        vim.lsp.config("lua_ls", {
            settings = {
                Lua = {
                    diagnostics = {
                        globals = {
                            "vim",
                        },
                    },
                },
            },
        })

        require("mason-lspconfig").setup({
            ensure_installed = {
                "lua_ls",
            }
        })

        vim.lsp.enable("lua_ls")
        vim.diagnostic.config({
            virtual_text = true,
            -- virtual_lines = true,
            update_in_insert = true,
        })

        vim.api.nvim_create_autocmd("LspAttach", {
            callback = function(args)
                local client = vim.lsp.get_client_by_id(args.data.client_id)

                if client and client.server_capabilities.diagnosticProvider then
                    vim.schedule(function()
                        vim.diagnostic.enable(true, {
                            bufnr = args.buf,
                        })
                    end)
                end
            end,
        })
    end,
}
```
接着修改bufferline.lua使bufferline也具备显示LSP的报错功能
```
return {
    "akinsho/bufferline.nvim",
    dependencies = {
        "nvim-tree/nvim-web-devicons"
    },
    opts = {
        options = {
            diagnostics = "nvim_lsp",
            diagnostics_indicator = function (_, _, diagnostics_dict, _)
                local indicator = " "
                for level, number in pairs(diagnostics_dict) do
                    local symbol
                    if level == "error" then
                        symbol = " "
                    elseif level == "warning" then
                        symbol = " "
                    else
                        symbol = " "
                    end
                    indicator = indicator .. number .. symbol
                end
                return indicator
            end
        }
    },

    keys = {
        { "<leader>bb", ":BufferLineCyclePrev<CR>", silent = true },
        { "<leader>bn", ":BufferLineCycleNext<CR>", silent = true },
        { "<leader>bp", ":BufferLinePick<CR>", silent = true },
        { "<leader>bd", ":bdelete<CR>", silent = true},
    },

    lazy = false
}
```
### 补全
在plugins下新建blink.lua，写入
```
return {
    "saghen/blink.cmp",
    version = '*',
    dependencies = {
        "rafamadriz/friendly-snippets"
    },

    event = "VeryLazy",
    opts = {
        completion = {
            documentation = {
                auto_show = true,
            },
        },
        keymap = {
            preset = "super-tab",
        },
        sources = {
            default = { "path", "snippets", "buffer", "lsp" },
        },
        cmdline = {
            sources = function()
                local cmd_type = vim.fn.getcmdtype()
                if cmd_type == "/" or cmd_type == "?" then
                    return { "buffer" }
                end
                if cmd_type == ":" then
                    return { "cmdline" }
                end
                return {}
            end,
            keymap = {
                preset = "super-tab",
            },
            completion = {
                menu = {
                    auto_show = true,
                },
            },
        },
    },
}
```
再修改basic.lua，追加写入
```
-- 如果查找的内容中不存在大写，则大小写不敏感
vim.opt.ignorecase = true
vim.opt.smartcase = true

-- 不要在查找之后继续高亮匹配结果
vim.opt.hlsearch = false
```