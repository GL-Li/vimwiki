### Ubuntu: install packages with apt (advanced package tool)

- Understand  `/etc/apt/sources.list` file

  This file lists repositories of packages from which Ubuntu can use `apt` to install packages. There are four types of repositories: main, restricted, universe, and multiverse for each Linux distribution (`$ lsb_release -a`)

  ```
  deb http://us.archive.ubuntu.com/ubuntu/ jammy main restricted
  ```

  A new Linux installation has  official repositories in the file. Users can add other respositories to this file, but are recommended to add them under `/etc/apt/sources.list.d`.

- Add repositories under directory `/etc/apt/sources.list.d`

  When install packages not in the repos listed in `sources.list` file, often a repo is added to directory `/etc/apt/sources.list.d` by default. For example, when I install `onedrive` package, a file `onedrive.list` is created which has one line of text

  ```
  deb [arch=amd64 signed-by=/usr/share/keyrings/obs-onedrive.gpg] https://download.opensuse.org/repositories/home:/npreining:/debian-ubuntu-onedrive/xUbuntu_22.04/ ./
  ```

- search packages by keywords with `apt-cache` .

  All information of packages in above repositories is cached in files in `/var/lib/apt/lists`.

  ```sh
  $ apt-cache search docx   # list packages related to docx file
  $ apt-cache search "pdf viewer"
  $ apt-cache show antiword # show more details of a package
  ```

  To view all packages stored in a file in `/var/lib/apt/lists`.

  ```sh
  $ vim /var/lib/apt/lists/us.archive.ubuntu.com_ubuntu_dists_jammy_main_binary-amd64_Packages
  ```

  The cached files in computer is updated to match online repository server with

  ```shell
  $ sudo apt update
  ```

  After update, we can upgrade packages that were installed with `apt` with

  ```shell
  $ sudo apt upgrade
  ```

- install packages

  ```shell
  $ sudo apt install pkg1 pkg2 pkg3
  ```

### Linux: delete packages and configurations

```shell
$ sudo apt purge xxxx # Remove a package completely
$ sudo apt autormove  # Remove dependencies that is not required by any other packages
$ sudo apt autoclean  # clean package .deb files saved in /var/cache/apt/archives/ but keep those not available on server
```

