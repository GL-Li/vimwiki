### HOW: how to set up Fedora after fresh installation?

- After installation - 7 Things You MUST DO After Installing Fedora Linux: https://www.youtube.com/watch?v=RrRpXs2pkzg
    - dnf config to speed up package installation
        - `sudo nana /etc/dnf/dnf.config` and add the block
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


