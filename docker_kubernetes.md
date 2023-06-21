## Outline
- Major reference
- Concept
- Workflow and SOP
- Minimal examples
- QA - quick answers to quick questions
- Raw notes

## Major references

- Youtube video: [Docker for R users Tutorial](https://www.youtube.com/watch?v=oehhZ98o6Zk)
    - materials saved at `~/OneDrive/learning-resources/docker-user2022-r-for-docker/`
     
- Udemy: Docker & Kubernetes: The Practical Guide, https://www.udemy.com/course/docker-kubernetes-the-practical-guide/

## Concept ====================================================================

### concept: volumes vs bind mounts

**summary**
    - use named volume to share data across containers
    - use bind mounts to share data between a container and the host computer

**why need them?**
    - For persistent data storage. Without volume, docker-generated data are stored in the container and destroyed with the removal of the container.
    - With volume and bind mounts, the data is stored on the host computer. Volumes and bind mounts are different mechanisms to store the docker-generated files.

**volume** is created and managed by docker engine on the host computer. Named volume is very useful to share data across containers.
    - annonymous volumes:
        - used before Docker version 1.9 when named volume was not available.
        - when started with `docker run --rm -v /container/dir ...`, the ananymous volume is removed when container is stopped.
        - created by docker engine and assigned with a randomly generated name,
        - can be checked with `docker volume` command,
        - can be create in Dockerfile or with command `docker run -v /docker/dir`.
    - named volumes:
        - good to share among containers. In the example below, contain1's `/input_data` and container2's `/output_data` are the same.
            - `$ docker run -v shared_volume:/input_data --name container1  image1`
            - `$ docker run -v shared_volume:/output_data --name container2 image2`
        - portable to other host computers comparing to bind mounts.
        - can only be created with `docker run -v abcdefg:/container/dir` command, where volume name abcdefg is a string, not a path.

**bind mounts** is managed by user
    - Good to share with other applications on the host
    - can be edited by users.
    - `docker run -v /host/dir/path:/container/dir/path`, all paths are absolute,  starting from root.

### concept: tmpfs vs writable layer

- why need them?
    - For temporary storage for this container

- writable layer: a thin layer on top of a container. All files to container directories without being mounted to host will be written here. These files co-exist with the container until the container is deleted.

- tmpfs mounts: mount host memory to a container directory. Deleted when the container is stopped. Keep in mind that tmpfs may exaust host memory.

### concept: protect container subdirectory from bind mounts

- what happened
    When we bind mount with `docker run -v /host/dir:/container/dir ...`, the first step is that the `/host/dir` overwrites `/container/dir`. Any subdirectories in the latter but not in the former are deleted.

- protect container subdirectories with annonymous volume
    - if the container has subdir_1 under `/container/dir/` but not under `/host/dir/`, we can add a second mount to the docker run
        - `docker run -v /host/dir:/container/dir -v /container/dir/subdir_1 --rm ...`
        - The rule is that container's subdirectories has priority over its parent directory over volume mounting. In this case subdir_1 is first mounted to an annonymous volume and will not be overwritten by the bind mounts.
        - remember to use `--rm` to delete the annonymous volume after stopping the container
    - alternatively, we can add a line in Dockerfile
        - `VOLUME /container/dir/subdir_1`
        - still need `--rm` in docker run.

## Workflow and SOP ===========================================================


### SOP: use host's gui to display graphic interface in docker container

    - `$ xhost + local:` to allow the docker using host's xserver. Run it if system restarted. Keep the `:` at the end of `local`.
    - `$ docker run -it --rm -e DISPLAY=$DISPLAY -v /tmp/.x11-unix:/tmp/.x11-unix --net host my/image` to start the container.
        - for example, `docker run -it --rm -e DISPLAY=$DISPLAY -v /tmp/.x11-unix:/tmp/.x11-unix --net host r-base R`, make a plot in R to display the figure in a pop up window.

### SOP: add user to  docker user group so no need to use `sudo` every time
https://askubuntu.com/questions/477551/how-can-i-use-docker-without-sudo

    - `sudo groups` to list all user groups, if there is no docker group
    - `sudo groupadd docker` to create docker user group. If it says docker group already exists, just ignore the step.
    - `sudo gpasswd -a $USER docker` to add current user to docker group
    - `newgrp docker` to activate the changes
    - ready to use docker command without sudo

### SOP: work with Dockerhub

    - create an account at dockerhub.com, lglforfun, standard 2nd class password
    - `docker login` from terminal. Will stay logged in.
    - `docker logout` to log out
    - names image as lglforfun/myimage:0.1.2 to store image to dockerhub account.
        - `docker push lglforfun/myimage:0.1.2` to push an image to Dockerhub.

## Minimal examples ===========================================================

### example: volume, mount local directory to container directory
This example mounts a local directory `$HOME/tmp` to a container, reads a text file `ttt.txt` from local directory and saves a text file `bbb.txt` to `$HOME/tmp`

- Build image. The directory contains two files: Dockerfile and test.py

    - build image named test_volume
        ```sh
        docker build . -t test_volume
        ```

    - Dockerfile
        ```
        FROM python

        WORKDIR /app

        COPY . /app

        RUN mkdir /data

        CMD ["python", "test.py"]
        ```

    - test.py
        ```
        # a local directory will be mounted to container's /data. ttt.txt is initially in local directory.
        g = open("/data/ttt.txt")
        print(g.read())

        # save bbb.txt to local directory
        with open("/data/bbb.txt", "w") as f:
          f.write("this has been done")

        print("file saved")
        ```

- Run a container. make sure `ttt.txt` exists in `$HOME/tmp/`. File `bbb.txt` is saved in `%HOME/tmp` after the run. Automatically remove the container after exiting the container with option `--rm` as this container only runs one time.
    ```sh
    docker run -v $HOME/tmp:/data --rm test_volume
    ```

## QA =================================================================

### QA: kubernetes pod with two containers

- https://kubernetes.io/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/

## QA: how to get into the terminal of running container?

- `$ docker exec -it bash` for Ubuntu based container
- `$ docker exec -it sh` for alpine-based container

## raw notes ==================================================================

### Docker: build images run push pull

- `$ docker build -t my-app:v1` create container images from a `Dockerfile`
- `$ docker images`  list all images repos, tags, and sizes
- `$ docker run -p 8080:8080 nginx`  create a container from an image
- `$ docker push my-app:v1`  stores image in a configured registry
- `$ docker pull nginx`  retrieve images from a configured registry

### Docker object

- `Dockerfile`: a text file that contains instructions needed to create an image.
    - `FROM`  defines base image
    - `RUN`   executes arbitrary commands
    - `CMD`  defines default command for container execution. Only one or the last one will be run.

- Docker image
    - Read-only template with instructions for creating Docker container
    - Build using instructions in a `Dockfile`. A new read-only image layer is created for each instruction.
    - A writeable layer is added when an image is run as a container.
    - Layers can be shared between images, which saves disk space and network badwidth.

- Container image naming
    - hostname/repository:tag  like rocker/tidyverse:4.2.1

- Docker containers
    - a runnable instance of an image
    - Can be created, stopped, started or deleted using the Docker API or CLI
    - Can connect to multiple networks, attach storage, or create a new image based on its current state
    - Is well isolated from other contianers or its host machine.

- docker networks, storage, and plugins
    - Networks are used for the isolated container communication
    - storage: Docker uses volumes and bind mounts to persist data even after a container stops
    - plugins: storage plugins provide the ability to connect ot external storage platforms.

### Docker: from Ubuntu

- install Docker Engine on Ubuntu
    - follow the instructions: https://docs.docker.com/engine/install/ubuntu/

- where is docker image stored
    - run `$ sudo docker info` and look for Docker Root Dir
    - The docker root directory is `/var/lib/docker` in Ubuntu, where all pulled images stored.
    - Do not need to do anything at docker root.

- using rocker/tidyverse:4.2.1
    - $ sudo docker pull rocker/tidyverse:4.2.1
    - $ sudo docker images       # list pulled images
    - $ sudo docker run rocker/tidyverse:4.2.1 R   # run image as a container and start R consol. Do not miss R at the end.

## 2023-03-06 Mon

### Docker: how to share docker images
https://www.howtogeek.com/devops/how-to-share-docker-images-with-others/

- share a `Dockerfile`, which may result in subtly different environment when others build image from it.

- share an pre-built image

    - share through Docker registry
        - the default registry is [Docker Hub](https://hub.docker.com/)
            - personal account, free, one private repository and unlimited public repositories only
            - pro account: $5 per month, unlimted private repositories
        - push to account `difodocker`
            - create a repo on Docker Hub. repo name my be lowercase alphanumeric such as `mytest`, which is also the name of the image
               - This step is not necessary. A repo will be created automatically when push from command line.
            - push a new version of image `mytest` to this repo
                ```sh
                # rename a local image to match the repo
                docker tag hello-world:latest difodocker/mytest:1.1.1

                # log into Docker Hub account
                sudo docker login -u difodocker -p xxxxx docker.io

                # push to docker hub, repo mytest will be created on Docker Hub automatically if not exists.
                docker push difodocker/mytest:1.1.1
                ```

                ```
                $ docker push my-account/my-image:v1.2.3
            ```
        - everyone can pull the image
            ```sh
            docker pull my-account/my-image:v1.2.3
            ```

## 2023-03-07 Tue

### Docker:
docker desktop: GUI on top of Docker engine, it is a Linux virtual machine

What is Docker:
    - A platform for OS-level virtualization
        - based on isolated containers
        - package diferent componenets of a sofware /app
        - enable seamless shipent and deployment
        - free and enterprise versions available

Workflow
    - project with a `Dockerfile`
        - `docker build` docker image
        - launch container
        - work

## 2023-03-08 Wed

### Docker dockerfile

- `FROM` - a base image hosted a registry
- `LABEL` - meta data for  the image to be build, such as maintainer, info, code repo in github, and licenxe
- `ARG` - variables available during build time only.
- `ENV` - environent variable, kept as a meta data in the image to be built.
- `RUN` - linux command, for example, `RUN mkdir scripts` to create a directory in the image to be built.
- `COPY` - copy files from current directory to a directory in the image, like `COPY *.R scripts`.
- `EXPOSE` - defines the port on the container to listen during run time, used for inter-container communication. It does not make the container accessible to the host. `docker run -p 1234:1234` is still required to get access to the host.
- CMD -
- ENTRYPOINT -
- VOLUME -
- WORKDIRECTORY -

## 2023-03-10 Fri

### Docker commands

#### Docker build
- `docker build . --progress=plain -t difodocker/test:0.01` to build image from current directory.
    - `$ man docker build` for more info
        - `-t/--tag` to rename an image.
        - `--progress=plian` to suppress exessive build message

- `docker build --file Dockerfile_2 -t gl/test1:0.0.1` to build from a specific dockerfile.

#### manage docker images
- `docker image ls` or `docker images` to list all local docker images
- `docker rmi image_id` to remove an image by its ID
    - `-f/--force` to force the removal if an image ID is referenced in two or more repositories.
- `docker image inspect image_id` to check the metadata of an image

#### docker run to initiate containers
- get into the terminal of a container. An ENTRYPOINT in Dockerfile may block this access.
    ```
    docker run --rm -it gl/test:0.0.2 bash
    ```
- pass an environment variables to a container in a run
    ```
    docker run -e "PI=3.14" --rm -it gl/learn_docker_cli:latest bash
    ```
- run comtainer with bind mounted volumes, `-v /path/to/loca/dir/:path/to/container/dir/`
    docker run does not accept `~` as user home directory, use `$HOME` or absolute path.
    ```
    docker run --rm -it -v /home/gl/aaaaa/:/home/work/ gl/learn_docker_cli bash
    ```
- run container with published ports, `-p host_port:container_port`.
    - Rstudio container has a default user name `rstudio` so its user home directory is `/home/rstudio`. A local directory should be mount under it.
        ```
        docker run -p 8787:8787 -e PASSWORD=1234 -v /home/gl/aaaaa:/home/rstudio/work/ docker_for_r/rstudio:v1.0
        ```
    - use `-e USER=gl` to change login user. The container's default `/home/rstudio/` is still available but `/home/gl/` is ok. If mounting a local directory to `/home/gl/`, the empty `rstudio` directory is visible.
        ```
        docker run -p 8787:8787 -e USER=gl -e PASSWORD=1234 -v /home/gl/aaaaa:/home/rstudio/work/ docker_for_r/rstudio:v1.0
        ```
## docker ps and stop: - list running containers and stop it
    ```
    docker run -d rocker/tidyverse:4.2.1   # -d run in background
    docker ps    # list running containers
    docker stop container_id   # stop a container

    # list all containers including stopped
    docker ps -a
    ```

## 2023-03-13 Mon

### docker exec: execute a command on an running container

To tell the command which container to use, assign the container a name with `--name=xxxx`.
    ```
    sudo docker run -d -p 8787:8787 -e PASSWORD=abcd --name=aaaa -v /home/gl/aaaaa:/home/gl/project docker_for_r/rstudio:v1.0
    # run bash of container aaaa
    sudo docker exec -it aaaa bash
    # stop a container by name
    sudo docker stop aaaa
    ````

## 2023-03-14 Tue

### docker-compose: run from yaml file

- install docker-compose if command not found
    ```sh
    sudo apt install docker-compose
    ```

- run container from a `docker-compose.yml` file
    ```sh
    docker-compose up
    ```

- a sample `docker-compose.yml` file
    ```
    services:
      my_rstudio:
        image: docker_for_r/rstudio:v1.0
        ports:
          - "8787:8787"
        environment:
          DISABLE_AUTH: "true"
          NEW_VAR: "foo"
        volumes:
          - type: "bind"
            source: "/home/gl/aaaaa"
            target: "/home/rstudio"
    ```

## 2023-03-15 Wed

### docker: develop in docker

The goal is to create a workflow that is easy to follow and share. Details in `~/OneDrive/learning-resources/docker-user2022-r-for-docker/04-docker-and-r/03-develop-in-docker`.

- create a `Dockerfile`, in which the versons of R package is controled by snapshot of Microsoft Cran. Example:
    ```
    FROM rocker/rstudio:4.2.0

    LABEL purpose="Example of developing in RStudio IDE"

    ENV MRAN_BUILD_DATE=2022-06-18

    RUN install2.r \
        -r https://cran.microsoft.com/snapshot/${MRAN_BUILD_DATE} \
        --error \
        data.table

    EXPOSE 8787
    ```

- create execcutable file to build image, for example `01-docker-build.sh` and run with `./01-docker-build.sh` from terminal to build a image.
    ```
    docker build . -t docker_for_r/rstudio:v1.0
    ```

- create an execcutable docker run file to create a container and run like `./02-run-docker.sh`. Each user needs to modify `YOUR_LOCAL_PROJECT_FOLDER` to match his computer.
    ```
     YOUR_LOCAL_PROJECT_FOLDER="/home/gl/aaaaa"

     docker run \
        --rm \
        -it \
        -p 8787:8787 \
        -e DISABLE_AUTH="true" \
        -v $YOUR_LOCAL_PROJECT_FOLDER:/home/rstudio \
        docker_for_r/rstudio:v1.0
    ```

- Open `http://localhost:8787` to work on the project.
    - Close RStudio when done
    - All work will be saved in `YOUR_LOCAL_PROJECT_FOLDER`.

- The three files, `Dockerfile`, `01-docker-build.sh`, and `02-run-docker.sh` can be shared with other users.

### R: package version control, microsoft cran, mran

The Microsoft CRAN time machine snapshot is shutting down. Need to find a replacement.

- miniCRAN R package: to create a local repository in a single foler and can be copied and shared. Not a good choice.

## 2023-03-16 Thu

### Docker: install packages, command line, renv

- install package from command line
    ```
    FROM rocker/rstudio:4.2.2

    LABEL purpose="Example of developing in RStudio IDE"

    # use remotes packages
    RUN Rscript -e "install.packages('remotes')"
    RUN Rscript -e "remotes::install_version('skedastic', version = '2.0.1')"

    # Example of a conventional `install.packages()` approach for a package version
    RUN Rscript -e "install.packages('https://cran.r-project.org/src/contrib/Archive/ggplot2/ggplot2_3.4.0.tar.gz', repos = NULL, type = 'source')"

    EXPOSE 8787
    ```

## 2023-03-17 Fri

### docker: WORKDIR the default directory where bash runs
Any RUB CMD ADD COPY and ENTRYPOINT in Dockerfile will be executed in this directory. The default WORKDIR is root /.

- example files: `OneDrive/learning-resources/docker-user2022-r-for-docker/04-docker-and-r/02-installing-r-packages/02c-using-renv-option-1`

- Dockerfile
    ```
    FROM rocker/r-ver:4.2.0

    LABEL purpose="Example of {renv}: Install packages when Docker image is built."

    ENV RENV_VERSION 0.15.5
    RUN R -e "install.packages('remotes', repos = c(CRAN = 'https://cloud.r-project.org'))"
    RUN R -e "remotes::install_github('rstudio/renv@${RENV_VERSION}')"

    # create work directory under root using /
    WORKDIR /renv

    # copy renv.lock from current directory into docker image's WORKDIR
    COPY renv.lock renv.lock

    # run Rscript from the terminal at WORKDIR
    RUN R -e "renv::restore()"
    ```

### docker: install linux packages

In Dockerfile, add the following commands. update and -y are required.

    ```
    RUN apt update
    RUN apt install vim -y
    ```

## 2023-03-20 Mon

### docker: deploy models
Example: `~/OneDrive/learning-resources/docker-user2022-r-for-docker/04-docker-and-r/04-deploy-models-in-docker`

- plumber package: to create a web API from existing R code.

    - `plumb-model.R` to genereate web API
        ```R
        message("Running plumb-model.R")

        library(plumber)

        lm_model <- readRDS("lm_model.rds")

        pr("predict.R") |>
            pr_run(port=8000, host="0.0.0.0")
        ```

    - `predict.R` used above
        ```R
        #* @param cyl cylinders
        #* @param disp displacement
        #* @param hp horsepower
        #* @post /predict
        function(req, cyl = 4, disp = 100, hp = 120) {

          message(sprintf("Running prediction function with {cyl: %s, disp: %s, hp: %s}",
          cyl, disp, hp))

          newdata <- data.frame(
                                cyl = as.numeric(cyl),
                                disp = as.numeric(disp),
                                hp = as.numeric(hp)
                                )

          predict(lm_model, newdata)
        }
        ```

- Dockfile based on `rocker/plumber`
    ```
    FROM  rstudio/plumber

    LABEL purpose="Example of model deployment in Docker"

    WORKDIR /plumber/
    COPY model/lm_model.rds lm_model.rds
    COPY model/plumb-model.R plumb-model.R
    COPY model/predict.R predict.R

    EXPOSE 8000

    ENTRYPOINT ["Rscript", "-e", "source('plumb-model.R')"]
    ```

## 2023-03-21 Tue

### docker: CMD and ENTRYPOINT

- CMD or ENTRYPOINT is the last line in Dockerfile.
- The command in CMD or ENTRYPOINT will be executed when run `docker run xxxx`.
    - The command in CMD can be overwritten in `docker run` and can be appended with new options.
    - The command in ENTRYPOINT cannot be overwritten, thus ensure the container is used as specified.
        - new options can be append to ENTRYPOINT command though.

## 2023-03-22 Wed

### docker: deploy Shiny app

- example: ~/OneDrive/learning-resources/docker-user2022-r-for-docker/04-docker-and-r/05-deploy-shiny-in-docker

- Sample Dockerfile
    ```
    FROM  rocker/shiny:4.2.0

    LABEL purpose="Example of shiny deployment in Docker"

    WORKDIR /home/app
    COPY my_app .

    # This command will run when Docker run -p 3838:3838 my/image:v1.0.
    # We still need option -p to open the port on the host.
    CMD ["R", "-e", "options(shiny.port = 3838, shiny.host = '0.0.0.0'); shiny::runApp('/home/app')"]
    ```

    Without the last CMD in the docker file, the docker container will be started with `sudo docker run -p 3838:3838 difodocker/shiny_deployment:v1.0 R -e "options(shiny.port = 3838, shiny.host = '0.0.0.0'); shiny::runApp('/home/app')"`

## 2023-04-12 Wed

- [x] docker: how to manage images and containers

    - images
        - tag
        - list
        - inspect
        - remove

    - containers
        - name
        - config
        - list
            - docker ps --help to view all options
            - docker ps   list all running containers
            - docker ps -a list all running and closed containers
                - docker start xxxx  restart a closed container by id or name. Unlike `docker run`, the container will run at the background.
        - remove

    - attached vs detached mode
        - by default, `docker run` starts a container in attached mode which keep the terminal actively associated to the container. The terminal logs what happens in the container.
            - to change the default use option `-d` to run container in detached mode
            - use `docker attach xxx` to reattach a running detached container.
        - in detached mode, the terminal is not associated to the container. This is the case when we `docker start` a closed container.
            - to attach a closed container, use `docker start -a xxx`, also include `-i` if want interactive mode.

## 2023-04-13 Thu

- [x] Docker: copy file from and to a running container

    - `Docker cp $HOME/tmp/ttt.txt container_name:/data`
    - `Docker cp container_name:/data/file1 $HOME/tmp`

## 2023-04-14 Fri

- [ ] Docker: share images
    - share Dockerfile, but may not always possible if you have installation requiring credentials.
    - share build image, for example, on DockerHub.

## 2023-04-16 Sun

- [ ] Docker: data and volume
    - annonymous volume: map the volume to a temporary directory in host, which is managed by docker
        - add the volume in Dockerfile with keyword VOLUME
        - to run it, `docker run -v volume ....`
        - docker volume ls, inspedct, ...

## 2023-04-17 Mon

- [x] docker: `.dockerignore` file, ignore files in COPY

- [x] docker: ARGument and ENVironment varaibles set in Docker file

    - ARG: only available when building image
        ```
        # docker build --build-arg DEFAULT_PORT=3000
        ARG DEFAULT_PORT=80
        ```
    - ENV: available in docker run.
        ```
        # default, can be changed in docker run --env PORT=3000 or
        # use --env-file and set environment variable in a file.
        ENV PORT 80
        EXPOSE $PORT
        ```

## 2023-04-18 Tue

- [ ] docker: container network

## 2023-04-19 Wed

- [x] Docker: Kubenetes minikube to simulate environment
    - install kubectl: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
    - install minikube: https://minikube.sigs.k8s.io/docs/start/
        - `$ minikube start` to download minikube use docker as driver. Many need Virtualbox if in Mac or Windows.
        - a docker image called `gcr.io/k8s-minikube/kicbase` is created
        - when started, a container named `minikube` is running.
        - `minikube status` to check status
        - `minikube dashboard` to start dashboard in browser. Add addons if propmt to do so.

## 2023-04-19 Thu

- [x] Docker: kubernetes objects

    -  two ways to create an kubernetes object
        - imperatively
        - declaratively

    - pod object
        - smallest unit Kubernetes can interact with
        - contains one or more container, shared resources like volumn, and cluster-internal IP, localhost
        - designed to be ephemeral: to be started, stopped, replaced as needed managed by Kubernetes.

    - deployment object
        - created by users
        - control one or more pods
            - set desired state
            - define which pod to run and how many instance
            - pause, delete, roll back
            - scaled dynamically

- [x] Docker: Kubernetes deployment

    - step 1: create docker images using docker build and push to Dockerhub.
    - step 2: start cluster with `minikube start` if not already
    - step 3: create a deployment object
        - `kubectl create deployment app_name --image=dockerhub_account/image_name`
        - `kubectl get depolyments` to check deployments
        - `kubectl get pods` to check pods
        - `minukube dashboard` to view the deployment.

## 2023-04-21 Fri

- [x] Docker: Kubernetes service object
    - expose pod to other pods or externally
    - group pos with a shared IP
    - create a service
        - `kubectl expose deployment app_name --type=LoadBalancer --port=1234` to create services
        - `kubectl get service` to view services
        - `minikube service app_name` to open the service in a browser
    - scale deployment
        - `kubectl scale deployment/app_name --replicas=3` to start 3 copy of the deployment

## 2023-04-27 Fri

- [x] Docker: multi-container application

## 2023-05-01 Mon

- [x] Docker: compose, `docker-compose.yml` file

    - when to use `-` in front of a line in yaml files: NO dash for `key: value` pair, dash for a list of single values, for example
        ```
        image: "mongo"          # key: value pair
        volumes:                # key:
            - data:/data/db     # - value
            - test:/data/test   # - another value
        ```

## 2023-05-02 Tues

- [ ] Docker: utility container that provide an environment

    - run container interactively but in detached mode
    - `$ docker exec -it container_name xxx`

### Section 4:  Networking: cross-container communication

**container to WWW communication** is as usual
    - simply use the normal www address such as `"https://swapi.dev/api/films"` as we do it locally.
    
**container to local host communication** use `host.docker.internal` as host domain address
    - local method is not working anymore. We cannot use localhost like in `'mongodb://localhost:27017/xxxxxxx'`.
    - replace `localhost` with `host.docker.internal` for example in
        - `mongodb://host.docker.internal:27017/xxxx`
        - `http://host.docker.internal:8787`
 
**container to container communication** by IPAddress
    - start the container `$ docker run -d --name mongodb mongo` using official mongo image
    - find its IPAddress under NetworkSettings by running `$ docker inspect mongodb`, which looks like `172.17.0.3` and is recoganized by the docker engine.
    - Above IPAddress can be used in other containers managed by the ssame docker engine such as in `mongodb://127.17.0.3:27017/xxxx`.
    - This is NOT a good way to communicate between containers as the IPAddress changes at each start and we have to revise the hardcode IPAddress each time.

**docker networker** is a better choice for container-to-container communication where container name is used as address
    - create a network named abcd with `$ docker network create abcd`
        - `$ docker network ls` to view all networks
    - start a container in the network: `$ docker run -d --name mongodb --network abcd mongo`
        - `$ docker network inspect abcd` to view containers in the network.
    - in other container in the same network, we can use the container name as the domain name, for example in `mongodb://mongodb:27017/xxxx`, where `27017` is the default port of mongodb. 

**docker network driver** default as **bridge**
    - `$ dockr network create --driver driver_name abcd` to specify driver for a docker network.
    - **bridge** driver allows container to find each other by container names
    - **host** driver for standalone container to share localhost network resources.
        - only works on Linux
        - The container uses host's IP address so port mapping is not required. Options `-p`, `--publish` are ignored.
        - better performance as it skips network address translation between container and host.
    - **overlay** driver allows communication between container on different hosts. Only works on swarm mode, which is deprecated.
    - **macvlan** to use MAC address
    - **third-party plugins** for additional functionalities

**standalone container with host network**
    - https://docs.docker.com/network/network-tutorial-host/
    - The network named `host` is always on when docker engine is started, so we do not need to manually create a docker network with "host" driver.
    - Simply specify `--network host` in `docker run ...`.

### Section 5: Building multi-container application with docker

**The question**: Given a project that runs locally, how to containerize it? In this case, we have a front-end JS page that connects to a back-end node server to read and write data from a database. The goal is to build three containers for front-end, back-end, and database respectively and run the project using the containers.

**Container 1: database**: we use mongoose database which has an official Docker image to run a container
    - `$ docker run --name mongodb --rm -d -p 27017:27017 mongo` where 27017 is the default port of the image.
    - This container works for local node application. Need change for node app container

**Container 2: node app**: we already have all the files that run locally. In this step, they are moved into a container. 
    - To build the docker image, we first need to create a Dockerfile
        ```dockerfile
        FROM node
        WORKDIR /app
        
        # install dependencies
        COPY package.json
        RUN npm install
        
        # moving in all files of the project to working directory
        COPY . . 
        
        # port that match what in app.js
        EXPOSE 80
        
        # run the app
        CMD [ "node", "app.js" ]
        ```
    - build image `$ docker build -t test/s6_backend` with name goals-node
    - 
    


