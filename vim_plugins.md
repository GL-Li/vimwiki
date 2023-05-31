# vimwiki

- cheatsheet: http://thedarnedestthing.com/vimwiki%20cheatsheet

## How to use Markdown syntex
- Add the following to .vimrc. Rename all `xxx.wiki` to `xxx.md` and change syntax to markdown syntax in all `xxx.md` files.

    ```
    let g:vimwiki_list = [{'path': '~/vimwiki/',
                          \ 'syntax': 'markdown', 'ext': '.md'}]
    let g:vimwiki_global_ext = 0
    ```

- All files saved in `~/vimwiki/` and can be edited directly.

- mapping
    - `\wl`: open link in vertical split
    - `\ww`: start vimwiki
    - `\wd`: delete an wiki file
    - `\wr`: rename an wiki file
