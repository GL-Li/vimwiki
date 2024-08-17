# NeoVim

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
