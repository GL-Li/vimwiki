# LunarVim

## Installation:

**To reinstall LunarVim**, 
    - first use uninstall script to uninstall. Follow instructions here: https://www.lunarvim.org/docs/installation
    - then delete `~/.config/lvim/` directory, which stores the setting of the last installation.

**Do not use a package manager to install NeoVim** as it may have default setting for Nvim which then LunarVim will have to modify, resulting in a crowed `config.lua` file. It is always good to start with clean config file.

**Install NeoVim from appImage** and create a soft link to a $PATH.
    - Download appImage from  https://github.com/neovim/neovim/wiki/Installing-Neovim
    - Extract and set path
        ```sh
        ./nvim.appimage --appimage-extract
        ./squashfs-root/AppRun --version

        # Optional: exposing nvim globally.
        sudo mv squashfs-root /
        sudo ln -s /squashfs-root/AppRun /usr/bin/nvim
        nvim
        ```
**Use nvm to install node and npm**
    - install `nvm` with `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash`. Ref: https://github.com/nvm-sh/nvm.
    - install LTS version nodejs with `nvm install --lts`
    - update npm with `npm install -g npm`

### install LunarVim on WSL of Windows

Remember to set `git config --global core.autocrlf input`. Vim clones plugins from github, which may have line ending issue.

## QA =========================================================================

### QA: how to autoformat code in LunarVim?

- Add `lvim.format_on_save = true` to `config.lua`. Code will be auto-formatted at save.

### QA: what are some most common LunarVim keybindings?

Below `M-` for key `Alt`, `C-` for key `Ctrl`, and `S-` for key `Shift`.
- `M-t` and `M-k` to move current line or selected lines up and down
- `C-` plus `jkhl` to move between windows
- `C-` plus `Down`, `Up`, left and right arrow to move the border of current window
- `C-t` toggle terminal
- `<leader>/` comment or uncomment current line or selected lines
- `<leader>e` to toggle file exploror
- `<leader>f` search for file
- `<leader>h` clear search highlight


### QA: how to enable clipboard in LunarVim to copy and paste from one lvim to another?

- `sudo apt install xclip` to install a third party clipboard provider as NeoVim does not provide one.
- add line `lvim.builtin.terminal.clipboard = true` to LunarVim's `config.lua` to enable the clipboard.
- Restart terminals and lvim and it's ready to `y` in one and `p` to another Lvim session.

### QA: how to setup chatGPT.nvim plugin for LunarVim?

- **Add chatGPT key to .bashrc** with a new line `export OPENAI_API_KEY="xh348f9afjf"`
- Add to `config.lua` inside `lvim.plugins = {}` to install chatgpt plugin and its dependencies.
    ```
    {
      "jackMort/ChatGPT.nvim",
        event = "VeryLazy",
        config = function()
          require("chatgpt").setup()
        end,
        dependencies = {
          "MunifTanjim/nui.nvim",
          "nvim-lua/plenary.nvim",
          "nvim-telescope/telescope.nvim"
        }
    }
    ```
- Start lvim and run `:ChatGPT` to start using it.

### QA: how to install nerd fonts in Windows for LunarVim?

Use Window Terminal installed from Windows store.

- Youtube: Beyond Code Live X: NerdFonts in Windows Terminal & WSL
    - go to `https://webinstall.dev/nerdfont/`. It is `nerdfont`, not `neerdfonts`.
    - select windows and copy the code for installation
    - open a Window terminal which is default at Powershell (get it from windows store, it is not powershell), paste the command, and run
        - if failed, just repeat it. Who knows.
    - close and reopen Windows terminal
        - from dropdown, select setting
        - select Ubuntu from left panel
        - scroll down and select Appearance
        - from Font face select `DroidSansMon NF` and save. NF stands for nerd fonts.
        - from dropdown select Ubuntu to start Ubuntu terminal. Nerdfonts works from this terminal, not from the WSL terminal.

### QA: how to close current buffer?

<space> c where space is the leader key

### Lunarvim nerd font - JetBrainsMono

- icons will not displayed correctly without Nerd fonts
- to install Nerd fonts
    - download JetBrainsMono Nerd Font from https://www.nerdfonts.com/font-downloads
    - unzip to /usr/share/fonts/JetBrainsMono, create `JetBransMono/` if not exists with
        - `$ sudo unzip JetBrainsMono.zip -d /usr/share/fonts/JetBrainsMono`
    - run `$ sudo fc-cache -fv` to add the new fonts to system, https://ostechnix.com/install-nerd-fonts-to-add-glyphs-in-your-code-on-linux/
    - check if added with `$ fc-list`

### Lunarvim for R programming
Only need to install Nvim-R plugin. All other plugins are already managed by Lunarvim, which much easier than Vim or NeoVim.

**add to Lunarvim's config.lua**
    ```lua
    lvim.plugins = {
        {"jalvesaq/Nvim-R"}
    }
    ```
**Turn off Nvim-R underscore to <- conversion** by adding `vim.g.R_assign = 0` to LunarVim's `config.lua` file.

**Nvim-R** key bindings

    - `\rf` to start R console

    - `\rq` quit R console

    - `\ro` open R object browser

    - `\d` to run a line

    - `\ss` run a selected block of code

    - `\aa` run all the script

    - `\xx` toggle comment

    - `:Rstop`: stop R excution

**R language server**: if r_language_server is not automatically detected in LunarVim after opening a `.R` file, add the following line to `config.lua`:
    ```lua
    -- require install.packages("languageserver") in R
    require'lspconfig'.r_language_server.setup{}
    ```
