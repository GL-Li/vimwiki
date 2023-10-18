## Docker containers

### docker: OneDrive for Linux

- Instructions: use Docker image https://github.com/abraunegg/onedrive/blob/master/docs/Docker.md
- Step-by-step
    - Turn off SELinux (Security Enhanced Linux, which will stop onedrive from running) by set `SELINUX=disabled` in `/etc/selinux/config`. This is the default setting for Fedora 38.
    - Create directory `OneDrive` in user home directory. Do not forget this step
    - Run `docker_onedrive` which is an executable saved in `~/bin/`. The file is:
        ```sh
        #!/bin/env bash
        
        export ONEDRIVE_UID=`id -u`
        export ONEDRIVE_GID=`id -g`
        export ONEDRIVE_DATA_DIR="${HOME}/OneDrive"
        mkdir -p ${ONEDRIVE_DATA_DIR}
        docker run -it --rm \
            --name onedrive \
            -v onedrive_conf:/onedrive/conf \
            -v "${ONEDRIVE_DATA_DIR}:/onedrive/data" \
            -e "ONEDRIVE_UID=${ONEDRIVE_UID}" \
            -e "ONEDRIVE_GID=${ONEDRIVE_GID}" \
            driveone/onedrive:edge
        ```
    - Authorize access to Microsoft Account by open the link in terminal and copy the address of the blank webpage after authentication.
    - `$ docker_onedrive` to start synch
    - `$ Ctrl p` then `$ Ctrl q` to exit after sync completed, but
    - the container is still running until `$ docker stop onedrive` or computer is restarted.

**Containerized tools**
    - vimwiki for all notes taking, at `~/bitbucket-repos/docker-vimwiki/`
    - rstudio for data analysis in R, at `~/bitbucket-repos/rstudio-docker`


## Other Tech Essentials

### tools: Bitbucket Pipelins

- https://blog.programster.org/bitbucket-pipeline-cheatsheet
- 

### tools: **Oracle Cloud always free tier**

- https://cloud.oracle.com/?region=us-ashburn-1
- Account:
    - Account name: liguanglai 
    - User name: liguanglai@gmail.com, 
    - pw: standard
- webpage virtual machine
    - Public IP: 150.136.25.2
    - User: ubuntu
    - t590 ssh public key added
    - `$ ssh ubuntu@150.136.25.2` to ssh login
- Tutorials
    - how to host a website on free tier: https://yoursunny.com/t/2020/OracleCloud-website/ 



**General purpose IDE tools**
    - LunarVim for all coding except for R data analysis.

**System and file maintainence tools**
    - btrassistant for btrfs file system
    - timeshift to backup system
    - unison to backup file
