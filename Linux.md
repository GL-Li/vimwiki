## Raw notes ==================================================================

### systemd
`systemd` is the first program to run when a Linux computer starts. It mamages all units, especially service ?????????????????????

**ref: youtube video**: [Systemd Deep-Dive: A Complete, Easy to Understand Guide for Everyone](https://www.youtube.com/watch?v=Kzpm-rGAXos) 
**Most commonly used systemd commands**
    - `$ systemctl status docker` to check the status of a service
    - `$ sudo systemctl start docker` to start a service
    - `$ sudo systemctl stop docker` to stop a running service
    - `$ sudo systemctl restart docker` to restart a stopped service with the same configuration
    - `$ sudo systemctl enable docker` to start the service when the computer starts
    - `$ sudo systemctl disable docker` to stop starting the service when computer starts
