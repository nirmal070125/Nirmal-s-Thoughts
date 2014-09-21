Copying a Docker Image from one host machine to another
=========

This post will explain how you could copy a [docker](https://www.docker.com/) image pulled to one host, to another host. This is a 3 step process;

1. Import the docker image as a file.

```sh
core@master ~ $ docker images
REPOSITORY                       TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
54.254.64.141:5000/stratos-php   latest              9499a33ebaa6        20 hours ago        700.3 MB
core@master ~ $ docker save -o stratos-php 9499a33ebaa6

```
This will import the docker image which has 9499a33ebaa6 as the id, to a file call stratos-php in your the current folder.

2. Now, you need to copy the 'stratos-php' file to the other host.

This can be done via 'scp' or some other mechanism. 

3. SSH into the other host and load the imported file as a docker image.

```sh
core@minion-1 ~ $ docker load -i stratos-php 
core@minion-1 ~ $ docker images
REPOSITORY                       TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
54.254.64.141:5000/stratos-php   latest              9499a33ebaa6        20 hours ago        700.3 MB

```
Note that the loaded image is also having the same image id as of the original.

That's it for now!
