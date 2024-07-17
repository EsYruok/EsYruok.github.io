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
# require("config.keymaps")
# require("config.options")
nvim lua/config/keymaps.lua
# local opts = {
    # noremap = true,
    # silent = true,
# }
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
vim.api.nvim_set_keymaps("n","T","<cmd>NvimTreeToggle<cr>",opts)
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
vim.api.nvim_set_keymaps("n","<C-h>","<cmd>BufferLineCyclePrev<cr>",opt)
vim.api.nvim_set_keymaps("n","<C-l>","<cmd>BufferLineCycleNext<cr>",opt)
vim.api.nvim_set_keymaps("n","<leader>bc","<cmd>bdelete %<cr>",opt)
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
vim.api.nvim_set_keymaps("n","<leader>ff","<cmd>Telescope find_files<cr>",opt)
vim.api.nvim_set_keymaps("n","<leader>fg","<cmd>Telescope live_grep<cr>",opt)
vim.api.nvim_set_keymaps("n","<leader>fh","<cmd>Telescope help_tags<cr>",opt)
vim.api.nvim_set_keymaps("n","<leader>fk","<cmd>Telescope keymaps<cr>",opt)
```

## Toggleterm.nvim
```sh
nvim lua/plugins/plugin-toogleterm.lua
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
vim.api.nvim_set_keymaps("t","<Esc>","<C-\\><C-n>",opt)
vim.api.nvim_set_keymaps("n","<leader>tf","<cmd>ToggleTerm direction=float<cr>",opt)
vim.api.nvim_set_keymaps("n","<leader>th","<cmd>ToggleTerm direction=horizontal<cr>",opt)
```

