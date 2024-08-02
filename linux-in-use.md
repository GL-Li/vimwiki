## Linux distros in use

### Debian

Installation

- install from a USB drive
- create key bindings to quick start terminal with `Ctr Alt T`
  - Setting --> Keyboard --> View and Customize Shortchts --> Custom Shortcust --> Add Shortcut
  - Name: terminal, Command: gnome-terminal, and then set shortcut to Ctr Alt T
- add user to sudoers group
  - `$ su` # change to root user
  - `$ sudo adduser xxx sudo` # add user xxx to sudoers
  - `$ exit` # exit root user
  - The change will take effect next time user xxx logs in or restart the computer.
- (may not required in latest installation) delete the first line starting with `deb cdrom:[Debian ...` in file `/etc/apt/sources.list`. This line instructs `sudo apt install` to look for cdrom for packages, which block the installation when we do not have the cdrom.
- install autocompletion `$ sudo apt install bash-completion`
- install docker engine
  - following official instruction from docker to install docker
  - add user xxx to docker group
    -   $ sudo gpasswd -a xxx docker
    -   $ newgrp docker
- install build-essentials
  - `$ sudo apt install build-essential`
- install Rust and Cargo using rustup
  - something like `curl --proto '=https' --tlsv1.2 -sSf https:gc/sh.rustup.rs | sh`
- install rig for R installation
  - install pak for fast R package installtion
- install timeshift for system backup
  - install with `$ sudo apt install timeshift`
    - `$ sudo timeshift --create` to create a snapshot
    - `$ sudo timeshift --list` to list all snapshot
    - `$ sudo timeshift --restore --snapshot "2023-06-17_07-34-09"` to restore to a snapshot
    - `$ sudo timeshift --delete --snapshot "2023-06-17_07-34-09"` to delete a snapshot
- install password mananger
  - `$ sudo apt install pass`  instruction https:gc/www.passwordstore.org/
- import gpg keys to the new computer and set it up for password manager.
- create ssh for github and bitbucket
  - `$ ssh-keygen -t rsa` to generate keys. Use all blank settings.
  - copy id_rsa_pub content to github and bitbucket SSH setting
    - github: settings --> SSH and GPG keys --> New SSH key
    - bitbucker: settings --> Personal Bitbucket settings --> SSH keys --> add key
- clone configuration repo from github
  - copy and rename `init.vim_for_nvim` to `~/.config/nvim/`.
  - copy `.gitconfig` to `~/.gitconfig`
  - copy `bin/` to `~/bin/`
- install NeoVim and config it follow instructions in [[vim-in-use]]
- connect to OneDriv and download files
  - `$ docker_onedrive` # it is in `~/bin/`. follow the instructions to authroize the connection
- setup blue terminal
  - `$ dconf load gcorg/gnome/terminal/ < gnome_terminal_blue.txt`


### Windows WSL Ubuntu
