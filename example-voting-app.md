## Source code
    - https://github.com/GL-Li/example-voting-app.git
    - `~/github-rerpos/forked-projects/sample-voting-app`
    - branch master for the original code
    - branch dev for my work on the original code

## Learning objectives
    - Run the app
    - Understand all code related to Docker in the project, with the help of ChatGPT
    - Adapt the code for my own needs

## Run the app

### docker compose up

**docker compose --file /path/to/compose/yaml up** to start containers specified in the ymal file. `--file /path/to/compose/ynal` cannot be ignored if a `docker-compose.yml` file is in the current directory.

**determine the right port in localhost**: the result container published several ports. When run `docker ps` we can see for this container a list of port: `0.0.0.0:5858->5858/tcp, :::5858->5858/tcp, 0.0.0.0:5001->80/tcp, :::5001->80/tcp`. We need to know which port is for view the results at localhost. How?
    - check the `README` file
    - `$ grep -r EXPOSE` to search for exposed ports in Dockerfiles. A good Dockerfile will explicitly expose the ports.
