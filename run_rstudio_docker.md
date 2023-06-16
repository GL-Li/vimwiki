
# RStudio docker image and container

**The goal** of this project is to create a RStudio docker image with all libraries installed and a bash script that can start a container in two ways:
    - `$ rstudio` and then open `localhost:8787/` in a browser. This container runs in host docker network without `-p` to publish port 
    - `$ rstudio 8765` and then open `localhost:8765/`. The container is publish to port 8765, or any other available port.
    - both case open files in current directory.

The project is at `bitbucket-repos/rstudio-docker/`.

## Docker image

**Dockerfile**

    ```dockfile
    FROM rocker/geospatial:4.3.0

    # install core packages from file. seperate 1 and 2 so 1 won't reinstall if
    # 2 changes
    COPY install_packages_1.R .
    RUN Rscript install_packages_1.R
    COPY install_packages_2.R .
    RUN Rscript install_packages_2.R

    # quick install of additiional packages
    #RUN Rscript -e "install.packages(c('DT', 'htmltools'))"

    # update packages when rebuild image
    RUN Rscript -e "update.packages(ask = FALSE)"

    EXPOSE 8787
    ```
    
## Run docker container

**run_rstudio.sh**
    ```bash
    #!/usr/bin/env bash

    # if number of positional parameters is 0, run the
    # container in host network
    if [[ $# -eq 0 ]] ; then
        docker run \
            -d \
            --rm \
            -e DISABLE_AUTH="true" \
            -v "$(pwd)":/home/rstudio \
            --network host \
            --name rstudio \
            lglforfun/rstudio_dev
    # if a port is given, publish the container to the
    # port and give a unique container name by port
    else
        docker run \
            -d \
            --rm \
            -e DISABLE_AUTH="true" \
            -v "$(pwd)":/home/rstudio \
            -p "$1":8787 \
            --name rstudio-"$1" \
            lglforfun/rstudio_dev
    fi
    ```
    
**create a soft link in $HOME/bin** which was added to the PATH
    - `$ ln /path/to/run_rstudio.sh rstudio`

**run from anywhere**
    - `$ rstudio` to start container rstudio at port 8787
    - `$ rstudio 8765` to start container rstudio-8765 at port 8765.

**to better use the RStudio server** create a directory `$HOME/r-projects` for all R projects and always start the container from this directory, which will given the same RStudio server settings.
