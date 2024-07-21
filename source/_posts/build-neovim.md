---
title: Neovim环境配置
categories: vim
abbrlink: 4081ca65
date: 2024-02-07 23:06:21
---
记录一下配置Neovim环境的过程. <!--more-->  

## Neovim
```sh
sudo pacman -S neovim
cd # home
mkdir .config/nvim && cd .config/nvim
touch init.lua
mkdir lua
mkdir lua/config
touch lua/config/keymaps.lua
touch lua/config/options.lua
nvim init.lua 
# require("config.options")
# require("config.keymaps")
nvim lua/config/keymaps.lua
# local opts = {
    # noremap = true,
    # silent = true,
# }
```

## Vim options
```sh
nvim lua/config/options.lua
```
```lua
vim.g.mapleader = " "
vim.g.maplocalleader = " "

vim.opt.clipboard = "unnamedplus"

--tab
vim.opt.tabstop = 4
vim.opt.shiftwidth = 4
vim.opt.expandtab = true
vim.opt.softtabstop = 4
vim.opt.shiftround = true

--ui
vim.opt.number = true
vim.opt.relativenumber = true
vim.opt.cursorline = true
vim.opt.colorcolumn = "160"

--search
vim.opt.incsearch = true
vim.opt.hlsearch = false
vim.opt.ignorecase = true
vim.opt.smartcase = true
```

## Lazy.nvim
```sh
sudo pacman -S luarocks ttf-jetbrains-mono-nerd
nvim lua/config/lazy.lua # https://lazy.folke.io/installation
nvim init.lua # require("config.lazy")
mkdir lua/plugins
nvim # :checkhealth lazy
```

## Nvim-tree.lua
```sh
nvim lua/plugin/plugin-nvim-tree.lua
```
```lua
return {
    'nvim-tree/nvim-tree.lua',
    version = "*",
    lazy = false,
    dependencies = {
        'nvim-tree/nvim-web-devicons',
    },
    opts = {
        actions = {
            open_file = {
                quit_on_open = true,
            },
        },
    },
}
```
```lua
--keymaps
vim.api.nvim_set_keymap("n","T","<cmd>NvimTreeToggle<cr>",opt)
```

## Bufferline.nvim
```sh
nvim lua/config/options.lua # vim.opt.termguicolors = true
nvim lua/plugins/plugin-bufferline.lua
```
```lua
return {
    'akinsho/bufferline.nvim',
    version = "*",
    dependencoes = { 'nvim-tree/nvim-web-devicons' },
    opts = {
        options = {
            offsets = {
                {
                    filetype = 'NvimTree',
                    text = 'File Explorer',
                    highlight = 'Directory',
                },
            },
        },
    },
}
```

> :h bufferline.nvim

```lua
--keymaps
vim.api.nvim_set_keymap("n","<C-h>","<cmd>BufferLineCyclePrev<cr>",opt)
vim.api.nvim_set_keymap("n","<C-l>","<cmd>BufferLineCycleNext<cr>",opt)
vim.api.nvim_set_keymap("n","<leader>bc","<cmd>bdelete %<cr>",opt)
```

## Lualine.nvim
```sh
nvim lua/plugins/plugin-lualine.lua
```
```lua
return {
    'nvim-lualine/lualine.nvim',
    dependencies = { 'nvim-tree/nvim-web-devicons' },
    opts = {
        options = {
            theme = 'jellybeans',
        },
    },
}
```

## Telescope.nvim
```sh
sudo pacman -S ripgrep fd
nvim lua/plugins/plugin-telescope.lua
nvim # :checkhealty telescope
```
```lua
return {
    'nvim-telescope/telescope.nvim',
    branch = '0.1.x',
    dependencies = { 'nvim-lua/plenary.nvim' },
}
```
```lua
--keymaps
vim.api.nvim_set_keymap("n","<leader>ff","<cmd>Telescope find_files<cr>",opt)
vim.api.nvim_set_keymap("n","<leader>fg","<cmd>Telescope live_grep<cr>",opt)
vim.api.nvim_set_keymap("n","<leader>fh","<cmd>Telescope help_tags<cr>",opt)
vim.api.nvim_set_keymap("n","<leader>fk","<cmd>Telescope keymaps<cr>",opt)
```

## Toggleterm.nvim
```sh
nvim lua/plugins/plugin-toggleterm.lua
```
```lua
return {
    'akinsho/toggleterm.nvim',
    version = '*',
    opts = {
        open_mapping = [[<C-r>]],
        direction = "float",
    },
}
```
```lua
--keymaps
vim.api.nvim_set_keymap("t","<Esc>","<C-\\><C-n>",opt)
vim.api.nvim_set_keymap("n","<leader>tf","<cmd>ToggleTerm direction=float<cr>",opt)
vim.api.nvim_set_keymap("n","<leader>th","<cmd>ToggleTerm direction=horizontal<cr>",opt)
```

## LSP
```sh
nvim lua/plugins/plugin-lsp.lua
```
```lua
return {
    --mason
    {
        'williamboman/mason.nvim',
        opts = {
            ui = {
                icons = {
                    package_installed = "✓",
                    package_pending = "➜",
                    package_uninstalled = "✗"
                },
            },
        },
    },
    --mason-lspconfig
    {
        'williamboman/mason-lspconfig.nvim',
        opts = {
            ensure_installed = { "lua_ls", "rust_analyzer" },
        },
    },
    --lspconfig
    {
        'neovim/nvim-lspconfig',
        config = function ()
            local lspconfig = require('lspconfig')
            lspconfig['lua_ls'].setup({})
            lspconfig['rust_analyzer'].setup({})
        end,
    },
    --lspsaga
    {
        'nvimdev/lspsaga.nvim',
        config = function ()
            require('lspsaga').setup({})
        end,
        dependencies = {
            'nvim-treesitter/nvim-treesitter',
            'nvim-tree/nvim-web-devicons',
        },
    },
}
```
```lua
--keymap
```

## Nvim-cmp
```sh
nvim lua/plugins/plugin-cmp.lua
```
```lua
return {
    {
        'L3MON4D3/LuaSnip',
        version = 'v2.*',
        build = 'make install_jsregexp',
        dependencies = {
            'saadparwaiz1/cmp_luasnip',
        },
    },
    {
        'hrsh7th/nvim-cmp',
        dependencies = {
            --'onsails/lspkind.nvim',
            'hrsh7th/cmp-nvim-lsp',
            'hrsh7th/cmp-buffer',
            'hrsh7th/cmp-path',
            'hrsh7th/cmp-cmdline',
            'neovim/nvim-lspconfig',
        },
        config = function ()
            local cmp = require('cmp')
            --local lspkind = require('lspkind')
            cmp.setup({
                snippet = {
                    expand = function(args)
                        require('luasnip').lsp_expand(args.body)
                    end,
                },
                mapping = cmp.mapping.preset.insert({
                    ['<C-b>'] = cmp.mapping.scroll_docs(-4),
                    ['<C-f>'] = cmp.mapping.scroll_docs(4),
                    ['<C-k>'] = cmp.mapping.select_prev_item(),
                    ['<C-j>'] = cmp.mapping.select_next_item(),
                    ['<C-Space>'] = cmp.mapping.complete(),
                    ['<C-e>'] = cmp.mapping.abort(),
                    ['<CR>'] = cmp.mapping.confirm({ select = true }),
                }),
                sources = cmp.config.sources({
                    { name = 'nvim_lsp' },
                    { name = 'luasnip' },
                },{
                    { name = 'buffer' },
                }),
                --lspkind
                --[[
                formatting = {
                    format = lspkind.cmp_format({
                        mode = 'symbol',
                        maxwidth = 50,
                        ellipsis_char = '...',
                        show_labeDetails = true,
                    }),
                },
                --]]
            })
            cmp.setup.cmdline({'/', '?'},{
                mapping = cmp.mapping.preset.cmdline(),
                sources = {
                    { name = 'buffer' }
                },
            })
            cmp.setup.cmdline(':',{
                mapping = cmp.mapping.preset.cmdline(),
                sources = cmp.config.sources({
                    { name = 'path' },
                },{
                    { name = 'cmdline' },
                }),
                mathcing = { disallow_symbol_nonprefix_matching = false },
            })
        end,
    },
}
```

