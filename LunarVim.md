# LunarVim

### install LunarVim on WSL of Windows

Remember to set `git config --global core.autocrlf=input`. Vim clones plugins from github, which may have line ending issue.


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
