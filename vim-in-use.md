## Install NeoVim

We will install neovim but use it as vim. We will use VSCode to cover the flexibility Neovim offers. The focus is to use it as a text editor.

All the configuration in `.vimrc` for Vim apply to NeoVim after making the following adjustment.

- Rename `.vimrc` to `init.vim` and copy to `~/.config/nvim/`.
- Instsall vim-plug to `~/.config/nvim/autoload/`:
    ```sh
    curl -fLo ~/.config/nvim/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
    ```
    
The easiest way to get the latest NeoVim is to download its appimage, make it executable, rename to `nvim` and copy to `$HOME/bin` (must in PATH). The latest NeoVim 0.10.1 has retrobox colorscheme and clipboard support.

In Windows WSL Ubuntu, which does not have FUSS to support appimage, extract the appimage and then make a soft link to the executable from `~/bin`: `$ ln -s ~/neovim/squarshfs-root/bin/nvim`. Hard link does not work as this `nvim` depends on files in `squarshfs-root/`.

This `init.vim` can be used by VSCode extension `VSCode Neovim` created by asvetliako. Unfortunately, `.vimrc` is not supported by VScode extension `Vim` by vscodevim. This is why we have this note.


    
## Understand vim

### vim key mapping

mapping keywords: `noremap`: non-recursive map for all mode:

- `nnoremap`: in normal mode
- `inoremap`: in insert mode
- `vnoremap`: in visual mode
- `cnoremap`: in command mode (when `:` is on)
- try to use it if possible to avoid infinite loop, for example:
    - `:map a b` and `:map b a` will cause an infinite loop. Pressing `a` activates `b`, then `a`, then `b`, ...
    - `:noremap a b` and `:noremap b a` will not. Pressing `a` activates `b` and then stop.

special keys

- <Esc>: Esc key
- <C>: Ctrl key
    - <C-j>: Ctrl j
- <CR>: Enter key
- <up>: up arrow key, similar for down, left right arrow keys
    - <C><up>: Ctrl up-arrow

map plugin's command

- `nnoremap <leader>, <Plug>(EasyAlign)ip*,<CR>`
