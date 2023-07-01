# LunarVim

## Installation:

- install `nvm` with `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash`. Ref: https://github.com/nvm-sh/nvm.
- Follow terminal instruction after installation to start using it
    ```
    [root@236f9c57ea50 /]# export NVM_DIR="$HOME/.nvm"
    [root@236f9c57ea50 /]# [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
    [root@236f9c57ea50 /]# [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
    ```
- install LTS version nodejs with `nvm install --lts`
- for Rust dependency when install LunarVim, install `sudo dnf groupinstall "Development Tools"`.

- QA
### install LunarVim on WSL of Windows

Remember to set `git config --global core.autocrlf=input`. Vim clones plugins from github, which may have line ending issue.

## QA =========================================================================

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
        - `$ sudo unzip JetBrainsMono.zip /usr/share/fonts/JetBrainsMono`
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

**Nvim-R** key bindings

    - `\rf` to start R console

    - `\rq` quit R console

    - `\ro` open R object browser

    - `\d` to run a line

    - `\ss` run a selected block of code

    - `\aa` run all the script

    - `\xx` toggle comment

    - `:Rstop`: stop R excution

**R language server protocol**: to add R lsp, open any R file in LunarVim, start R with `\rf`, then from the R console `install.packages("languageserver")`. Check for dependencies at https://github.com/REditorSupport/languageserver/.
