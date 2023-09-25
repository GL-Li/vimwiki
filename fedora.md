### HOW: how to set up Fedora after fresh installation?

- After installation - 7 Things You MUST DO After Installing Fedora Linux: https://www.youtube.com/watch?v=RrRpXs2pkzg
    - dnf config to speed up package installation
        - `sudo nano /etc/dnf/dnf.config` and add the block
            ```
            fastestmirror=True   # find the fastest mirror to download. Take some time to begin but will be fast later on
            max_parallel_downloads=3  # allow downloading 3 packages the same time, depending on internet speed
            defaultyes=True   # do not ask before download, always yes, like -y in Ubuntu
            keepcache=True    # to remove cached files, run $ sudo dnf clean all
            ```
    - `sudo dnf update` to update system
    - enable RPM fusion for more package. From https://rpmfusion.org/Configuration 
        - copy command under **Fedora with dnf**
            - `$ sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm` 
        - scroll down to get AppStrem metadata with `$sudo dnf groupdate core`. 
    - enable flathub for flatpak packages from https://flatpak.org/setup/Fedora with `$  flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo`.
    - optional - change hostname which is defaulted at `fedora`. To change
        - `$ sudo hostnamectl set-hostname t590`
    - enable bash autocompletion: `$ sudo dnf install bash-completion`

- install development tools: `$ sudo dnf groupinstall "Development Tools"`

- create shortcut: settings --> Keyboard --> Keyboard Shorts at the bottom --> Custom Shortcuts at bottom --> click on + need to know the actual command 
    - map terminal to `Ctrl Alt t`: command is `gnome-terminal`
- Useful builtin shortcut: Windows-a to show all app
- install btrfs-assistant: `$ sudo dnf install btrfs-assistant`

- set up and install my faverate tools:
    - generate ssh key for github and bitbucket:
        - `$ ssh-keygen -t rsa`
        - `$ cat .ssh/id_rsa.pub`
    - copy `.bashrc` for Fedora and `~/bin/` directory
      setup terminal configuration with `$ dconf load /org/gnome/terminal/ < /path/to/configurations/gnome_terminal_setting_fedora.txt`. Select font to Monospace if new terminal looks not good.
    - install Docker engine and bring in vimwiki container:
        - google install docker fedora to get the latest installation method
        - `$ sudo systemctl enable docker` to start docker engine automatically when Linux starts.
        - add user to docker group
    - install Rust, which is used by LunarVim later on
        - use rustup to install, google rustup install and copy paste the command
        - restart terminal and check installation with `$ rustc --version` and `$ cargo --version`.
    - install LunarVim:
        - `$ sudo dnf install neovim`
        - install nvm from github with curl
        - install laxygit
        - install LunarVim
        - install JetBrainsMono Nerd fonts, reboot to take effects.
        - copy `config.lua`
    - install R and RStudio:
        - `$ sudo dnf install R`
            - The cran2copr project maintains binary RPM repos for CRAN -- need to check if worth it??????????????
            - More on Fedora Packages of R Software Cran project , read ?????????????????????????????
        - `$ sudo dnf install rstudio-desktop`
    - give user permission to `/mnt/` to mount discs:
        - `$ sudo chown -R $USER /mnt` to change owner of `mnt` to current user. If `/mnt` has subdirecotries, add `-R` to change owner recursively.
        - `$ sudo mount /dev/sda1 /mnt/d`. Create `d` subfolder if not existe.
    - mount disk:
        - `$ sudo mount /dev/sda1 /mnt/d` to mount
        - add to `/etc/fstab` to auto-mount at reboot:
            - `$ ls -al /dev/disk/by-uuid/` to list the uuid of each parition.
            - add `/dev/sda` to `/etc/fstab` by uuid like 
              `uuid=1c23fddafar-j3fd-fda4-3f5difjdas45 /mnt/e  ext4  default  0 0`
            - `$ sudo systemctl daemon-reload` to load the changes
            - `$ sudo findmnt --verify` to make sure no error reported
            - reboot

### Fedora default shortcut

**Terminal**
- Windows and tabs
    - `Ctr Shift N` to start a new terminal --- use `Ctrl Alt T` if the it used by other applications such Google Chrome
    - `Ctr Shift T` to add a new tab to current terminal
    - `Ctr Shift W` to close a tab
    - `Ctr Shift Q` to close terminal window
    - `Alt 3` to switch to tab 3
    - In **Terminal Preference - general**, set new tab position to "Next" instead of last
    - Make sure line `source /etc/profile.d/vte.sh` is in `.bashrc`, which opens new tab in current directory
- Search and find in terminal
    - `Ctr Shift F` search in the window, `Esc` to close the search box
    - `Ctr Shit J` to clear search highligh
- 


