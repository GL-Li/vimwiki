# Docker image of vim with vimwiki

The goal is to build a docker image that includes vim and vimwiki. It will be used as a note-taking tools in all my computers.

## references

- The project is adapted from an github project: https://github.com/antoniopantaleo/docker-vim.git, which build a vim docker image from alpine linux base image.
- The project is on https://GL-Li@bitbucket.org/gl-li/docker-vimwiki.git


## Modification

### tech notes
- to get into alpine linux shell, run `sh` instead of `bash`.
- 

### Dockerfile
- use `$ curl` to download `plug.vim` into `autoload/` so `vim-plug` will be loaded when vim is started.
- use `$ vim` to open `vimrc` and run vim commands from terminal
- default vimwiki working directory is `/root/vimwiki/`, which matches the default ones in all host computers.
- the final Dockerfile

    ```
    FROM alpine:latest
    RUN apk update && apk add git curl vim
    # download vim-plug to a specific directory, need to modify base on vim version
    RUN curl -fLo /usr/share/vim/vim90/autoload/plug.vim https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
    RUN apk del curl
    RUN rm /etc/vim/vimrc
    WORKDIR /etc/vim
    COPY . .
    
    # vim command line options
    # -E improved exit mode
    # -s start silently
    # -u use /etc/vim/vimrc to initialize vim (open vimrc in vim)
    # -i NONE viminfo.file used 
    # -c run command :PlugInstall then :qa
    RUN vim -Es -u /etc/vim/vimrc -i NONE -c "PlugInstall" -c "qa"

    # create default directory for vimwiki and set it as working directory
    RUN mkdir /root/vimwiki
    WORKDIR /root/vimwiki

    ENTRYPOINT [ "vim" ]
    ```
    
### vimrc
- No dot before `vimrc`. It is called explicitly in Dockerfile.
- only plugin `vimwiki` is installed. Othe settings follow my local `.vimrc`.
- the final vimrc
    ```
    set nocompatible  " always not compatible with vi
    syntax on

    " plugins
    call plug#begin()
    Plug 'vimwiki/vimwiki'

    " autocompletion
    call plug#end()

    " vimwiki markdown syntex
    let g:vimwiki_list = [{'path': '~/vimwiki/',
                          \ 'syntax': 'markdown', 'ext': '.md'}]
    let g:vimwiki_global_ext = 0

    " display settings
    set encoding=utf-8 " encoding used for displaying file
    set ruler " show the cursor position all the time
    set noshowmatch " not highlight matching braces
    set showmode " show insert/replace/visual mode
    set number relativenumber " show line numbers and relativenumber
    set wildmenu  " show

    " write settings
    set confirm " confirm :q in case of unsaved changes
    set fileencoding=utf-8 " encoding used when saving file
    set nobackup " do not keep the backup~ file

    " edit settings
    set backspace=indent,eol,start " backspacing over everything in insert mode
    set expandtab " fill tabs with spaces
    set nojoinspaces " no extra space after '.' when joining lines
    set shiftwidth=4 " set indentation depth to 8 columns
    set softtabstop=4 " backspacing over 8 spaces like over tabs
    set tabstop=4 " set tabulator length to 8 columns

    " search settings
    set hlsearch " highlight search results
    set ignorecase " do case insensitive search...
    set incsearch " do incremental search
    set smartcase " ...unless capital letters are used

    " file type specific settings
    filetype on " enable file type detection
    filetype indent on " automatically indent code

    " syntax highlighting
    set t_Co=256
    set t_ut=
    colorscheme slate " set color scheme, must be installed first
    set background=dark " dark background for console
    syntax on " enable syntax highlighting

    " characters for displaying non-printable characters
    set listchars=eol:$,tab:>-,trail:.,nbsp:_,extends:+,precedes:+

    " softwrap text
    set linebreak
    set formatoptions=ro
    set comments=b:-
    set breakindent
    set autoindent
    set breakindentopt=shift:2
    "
    "
    " general key mappings
    " ==============================================================================

    " press F8 to turn the search results highlight off
    noremap <F8> :nohl<CR>
    inoremap <F8> <Esc>:nohl<CR>a

    " switch between windows
    nnoremap <C-H> <C-W>h
    nnoremap <C-J> <C-W>j
    nnoremap <C-K> <C-W>k
    nnoremap <C-L> <C-W>l

    " vimwiki
    nnoremap <leader>wl :VimwikiVSplitLink<CR>

    " quit
    nnoremap <leader>q :q<CR>
    ```
    
## build and run

### build.sh
- run `$ ./build.sh` to build the image
    ```
    docker build . -t lglforfun/vimwiki
    ```
    
### vimwiki.sh
- the file
    ```
    #! /usr/bin/env bash
    docker run --rm -it -v $HOME/vimwiki:/root/vimwiki lglforfun/vimwiki
    ```
- create a softlink in `~/bin/`, which is in `$PATH` with command `$ ln -s path/to/vimwiki.sh vimwiki` so that we can start a docker container from anywhere.
- run `$ vimwiki` then `\ww` to edit file at `~/vimwiki/`.
