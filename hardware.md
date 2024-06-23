## GPU - NVIDEA GTX 1060
https://wiki.debian.org/NvidiaGraphicsDrivers

### Detect installed GPU

Run
`$ lspci -nn | egrep -i "3d|display|vga"`
or 
`$ nvidia-detect`, which is installed from a `deb` file downloaded from https://packages.debian.org/bookworm/amd64/nvidia-detect/download
GTX 1060 is supported by all version of drivers

### Install drivers

Debian 12 has precompiled version 525.105.17, which support GTX 1060. Following the following precedures to install.

**switch to root user**

- `$ sudo su`

**Kernel headers**

- `# apt install linux-headers-amd64` where `#` indicates the root user.

**Install for Debian 12**

- Add the following line to `/etc/apt/sources.list`
    ```
    deb http://deb.debian.org/debian/ bookworm main contrib non-free non-free-firmware
    ```
- Update and install
    ```
    # apt update
    # apt install nvidia-driver firmware-misc-nonfree
    ```
- Restart computer: take a while, will see black screen for a while before and after logging in.

**Configuration**

Debian 12 should configure it automatically.

**CUDA**

- `# apt install nvidia-cuda-dev nvidia-cuda-toolkit`

**OptiX**

- `# apt install libnvoptix1`

### Activate GPU card

If `$ lspci -nn | egrep -i "3d|display|vga"` returns more than one lines, that means the computer uses a hybrid graphics chipset, we may need to activate the installed GPU for 3D effect following instruction https://wiki.debian.org/NVIDIA%20Optimus.

**Identification**

- `$ lspci | grep -E "VGA|3D"` check GTX 1060 against the list at https://www.nvidia.com/en-us/geforce/technologies/optimus/supported-gpus/.
- GTX 1060 desktop is not in the list, so do nothing.
