
## Outline

- Major reference
- Workflow and SOP
- Minimal examples
- Glossaries
- Raw notes

## Raw notes ==================================================================

- [ ] Youtube video: https://www.youtube.com/watch?v=GazrDjcdeG4

- [ ] julia-vim
    - https://github.com/JuliaEditorSupport/julia-vim
    - It helps edit Julia in vim, not runs Julia code
    - `:h julia-vim` for documentation.

- [ ] LunarVim for Julia
    - https://github.com/davibarreira/LunarVimConfig
    - LunarVim is a fork of NeoVim
    - Julia is native to LunarVim, with minumal configuration.
    - installation: https://www.lunarvim.org/docs/installation
    - install Julia for LunarVim: https://www.lunarvim.org/docs/features/supported-languages/julia
    - plugins are managed in ~/.config/lvim/config.lua: https://www.lunarvim.org/docs/master/configuration/plugins

- [ ] Nerd fonts for LunarVim
    - icons will not displayed correctly without Nerd fonts
    - to install Nerd fonts
        - download JetBrainsMono Nerd Font from https://www.nerdfonts.com/font-downloads
        - unzip to /usr/share/fonts/JetBrainsMono, create JetBransMono if not exists
        - run `$ sudo fc-cache -fv` to add the new fonts to system, https://ostechnix.com/install-nerd-fonts-to-add-glyphs-in-your-code-on-linux/
        - check if added with `$ fc-list`

- [ ] work at REPL
    - `>?` to enter help and then type function name
    - `>;` to ener shell terminal, and `ctrl c` to get back to Julia REPL
    - `>]` to enter package manager, `ctrl c` to get back to REPL when `pkg>` is at insert mode.
        - `pkg> st` to see status of installed packages
        - `pkg> up` to update installed packages
        - `pkg> add pkg_name` to add a new package
        - `pkg> rm pkg_name` to remove a installed package.
    - `> include("test.jl") to run a file
        - or `$ julia test.jl` from terminal
    - `> \alpha` then tab key to show greek letter. The unicode mode also works in editor.
    - `> x\_1 TAB` for x1 subscription

- [ ] common functions
    - `typeof()` to check varaible data type

- [ ] data type
    - type hierarchy, 587 datatypes
        - view with `subtypes(Any)`
        - subtypes(Number)
        - supertypes(Number)
        - Int64 <: Int to check if subtype
    - `(3 + 5)::Int` to make sure the expression output is of the right datatype.

- [ ] constant whose type cannot be changed, value can be changed but with a warning
    ```
    const AAA = 42

    # error
    AAA = "abcd"

    # warning
    AAA = 99
    ```
