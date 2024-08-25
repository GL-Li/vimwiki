# NeoVim

## NeoVim configuration
https://www.youtube.com/watch?v=zHTeCSVAFNY&list=PLsz00TDipIffreIaUNk64KxTIkQaGguqn&index=1

### EP1 From 0 to IDE NeoVim from scratch

- convert vimrc settings to lua
    - use `vim.cmd("set tabstop=2")` to wrap vimscript setting
    - use `vim.cmd.colorscheme` "xxx" to setup colorscheme
- use `lazy.vim` to manage packages

```lua
vim.cmd("set expandtab") -- for vimrc set command
vim.cmd("set tabstop=2")
vim.cmd("set softtabstop=2")
vim.cmd("set shiftwidth=2")
vim.g.mapleader = " " -- for global settings

-- install lazy.vim if not already
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not (vim.uv or vim.loop).fs_stat(lazypath) then
  local lazyrepo = "https://github.com/folke/lazy.nvim.git"
  local out = vim.fn.system({ "git", "clone", "--filter=blob:none", "--branch=stable", lazyrepo, lazypath })
  if vim.v.shell_error ~= 0 then
    vim.api.nvim_echo({
      { "Failed to clone lazy.nvim:\n", "ErrorMsg" },
      { out, "WarningMsg" },
      { "\nPress any key to exit..." },
    }, true, {})
    vim.fn.getchar()
    os.exit(1)
  end
end
vim.opt.rtp:prepend(lazypath) -- add to rtp - runtimepath

local plugins = {
  { "catppuccin/nvim", name = "catppuccin", priority = 1000 },
  {
    'nvim-telescope/telescope.nvim', tag = '0.1.8',
    dependencies = { 'nvim-lua/plenary.nvim' }
  },
  {"nvim-treesitter/nvim-treesitter", build = ":TSUpdate"},
}
local opts = {}

-- load module "lazy" and setup configuration
require("lazy").setup(plugins, opts)

-- load module "telescope.builin" and remap its functions
local builtin = require('telescope.builtin')
vim.keymap.set('n', '<C-p>', builtin.find_files, {})
vim.keymap.set('n', '<leader>fg', builtin.live_grep, {}) -- sudo apt install ripgrep

-- setup nvim-treesitter configuration
local config = require("nvim-treesitter.configs")
config.setup({
  ensure_installed = {"lua", "python", "r", "rust"},
  highlight = {enable = true},
  indent = {enable = true},
})

-- load catppuccin and use it as colorscheme
require("catppuccin").setup()
vim.cmd.colorscheme "catppuccin" -- for colorscheme
```


### EP2 
Split the big `init.lua` into multiple files in a structured way, which makes it easy to add new plugins. Each plugin and its configuration is a xxx.lua file under `lua/` directory.

```
.
├── init.lua
├── lazy-lock.json
└── lua
    ├── plugins
    │   ├── catppuccin.lua
    │   ├── lualine.lua
    │   ├── neo-tree.lua
    │   ├── telescope.lua
    │   └── treesitter.lua
    └── vim-options.lua
```

### EP3 manson and lsp
`mason-lspconfig.lua` to setup lsp

```lua
return {
  {"williamboman/mason.nvim",
    config = function()
      require("mason").setup()
    end
  },
  {
    "williamboman/mason-lspconfig.nvim",
    config = function()
      require("mason-lspconfig").setup({
        ensure_installed = {
          "lua_ls", "pyright", "r_language_server", "rust_analyzer", "bashls"
        }
      })
    end
  },
  {
    "neovim/nvim-lspconfig",
    config = function()
      local lspconfig = require("lspconfig")
      lspconfig.lua_ls.setup({})
      lspconfig.pyright.setup({})
      lspconfig.r_language_server.setup({})
      -- :help vim.lsp.but for more info
      -- place cursor on a function and press shift k to display the documentation of 
      -- the function, disappear if cursor moved away
      vim.keymap.set('n', 'K', vim.lsp.buf.hover, {})
      -- go to definition, more can be found here 
      -- https://github.com/neovim/nvim-lspconfig/blob/master/test/minimal_init.lua
      vim.keymap.set('n', 'gd', vim.lsp.buf.definition, opts)
      -- code action to suggest corrections of code issue
      vim.keymap.set('n', '<leader>ca', vim.lsp.buf.code_action, {})
    end
  },
}
```

### EP4 linter and formatter
Need to install formatters and linters using `:Mason`. Remember to select right linter and formatter for specific programming languages and set it up with non-ls plugin.

```lua
-- linting and formatting
return {
  "nvimtools/none-ls.nvim",
  config = function()
    local null_ls = require("null-ls") -- still use null-ls
    null_ls.setup({
      sources = {
        null_ls.builtins.formatting.stylua, -- need to install stylua from :Mason
        null_ls.builtins.completion.spell,
      },
    })

    -- map format key to reformat the code
    vim.keymap.set("n", "<leader>gf", vim.lsp.buf.format, {})
  end,
}
```


### EP5 autocompletion and code snippets



## LazyVin

To install a lsp, use Mason by `<leader>cm` to start Mason and look for the lsp for a language, press `i` to install the lsp.

## AstroNvim

- install nerd fonts using getnf and select jetBrainMono, and restart computer for it to take effec
- install following github instructions, ignore option part at the moment
- start nvim, may be two or more times for all plugins to automatically installed.
- install lsp from within nvim
  - :LspInstall pyright
  - :LspInstall r-language-server
  - :LspInstall rust_analyzer
  - :LspInstall awk_ls
  - :LspInstall bashls
  - :LspInstall markdown-oxide

## Lua basics for NeoVim

### require function

**load .lua file** in `config.lua` or `init.vim`:
- `require("myfile')` runs file `myfile.lua` under directory `~/.config/nvim/lua/myfile.lua`. No extension in the file called by `reauire` function.
- `require('other_files/myfile2')` to run file at `~/.config/nvim/lua/other_files/myfile2.lua`. It equals to `require("other_files.myfile2")`, which use dot instead of slash.
- if a directory contains a `init.lua` file, for example, `~/.config/nvim/lua/module_1/init.lua`, it can be called with `require("module_1")`, without specify `init`.

### run vim command from Lua

**examples** using `vim.cmd`
    ```lua
    # as quoted string
    vim.cmd("colorschemem habamax")
    vim.cmd("%s/foo/bar/g")
    
    # using literal string in double bracket
    vim.cmd([[colorschemem habamax]])
    vim.cmd([[%s/foo/bar/g]])
    vim.cmd([[
        highlight Error guibg=red
        highlight link Warning Error
    ]])
    
    # or use dot
    vim.cmd.colorscheme("habamax")
    vim.cmd.hightlight({ "Error", "guibg=red" })
    ```

### call vim functions from Lua

**example** using `vim.fn`
    ```lua
    print(vim.fn.printf("Hello from %s", "Lua"))
    
    local reversed_list = vim.fn.reverse({'a', 'b', 'c'})
    print(vim.inspect(reversed_list))   -- print 'c', 'b', 'a'
    ```

### variables

- `vim.g` global varaibles `g:`
- `vim.env` environment variables defined in the editor session

### set options in Lua

- `vim.opt.smarttab = true` equals `:set smarttab`
- `vim.opt.smarttab = false` equals `:set nosmarttab`
- `vim.opt_global` = `:setglobal`
- 


### install python and python notebook plugin for neovim

https://github.com/dccsillag/magma-nvim, remember to use key mappings that do not conflict with nvim-R.
https://www.youtube.com/watch?v=wzrZPcwh-bE


## QA ======================================================================== 

### QA: how to install neovim using appimage?

- copy `nvim.appimage` executable and rename it to `/usr/local/bin/nvim`.
- save `config.lua` in `~/.config/nvim/`.
