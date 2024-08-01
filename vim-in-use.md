## Important note
All the configuration in `.vimrc` for Vim apply to NeoVim after making the following adjustment.

- Rename `.vimrc` to `init.vim` and copy to `~/.config/nvim/`.
- Instsall vim-plug to `~/.config/nvim/autoload/`:
    ```sh
    curl -fLo ~/.config/nvim/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
    ```
    
The easiest way to get the latest NeoVim is to download its appimage, make it executable, rename to `nvim` and copy to `$HOME/bin` (must in PATH). The latest NeoVim 0.10.1 has retrobox colorscheme and clipboard support.

This `init.vim` can be used by VSCode extension `VSCode Neovim` created by asvetliako. Unfortunately, `.vimrc` is not supported by VScode extension `Vim` by vscodevim. This is why we have this note.


## Get vim
Two must have vim features:

- retrobox theme
    - built-in 
    - or from plugin `vim/colorschemes` if not avaible in vim
- clipboard support

Ways to get vim

- install vim-gtk3 if possible
- build from source


### build vim with clipboard

- install dependencies:
    ```sh
    sudo apt-get update
    sudo apt-get install -y build-essential ncurses-dev python3-dev ruby-dev lua5.1 liblua5.1-dev
    sudo apt-get install -y xclip xsel  # For clipboard support
    ```
- clone the vim source code from github
- config to enable clipboard. `--enable-gui-auto` is for clipboard.
    ```sh
    ./configure --with-features=huge --enable-multibyte --enable-python3interp --enable-rubyinterp --enable-luainterp --enable-gui=auto    
    ```
- build and install
    ```sh
    make
    sudo make install
    ```
- Start a new terminal, verify installation and look for `+clipboard`.
    ```sh
    vim --version
    ```
    
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
