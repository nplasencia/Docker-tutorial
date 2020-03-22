<p align="center">
  <img src="https://cdn.worldvectorlogo.com/logos/docker.svg" width="250px"/>
</p>

# Docker

It will help us with the automation of our deploys using _containers_. This adds control and security to our deploys.

## 1. Installation

[Linux](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-engine---community) - [Mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac)

## 2. Concepts

### Images:

Read only templates used to create containers. They are created using the commands `docker_build` or `docker_commit`.

The images are created using other images.

They are saved in a _docker registry_.

### Containers:

If we think in an OOP class, the __container__ is the instance of an image. The containers are a light and portable encapsulation of an environment where our applications are executed.

### Repositories:

It is a collection of different images with the same name but different tag. Usually, each tag represents a different version of the image.

### Registries:

A __registry__ is the place where we have stored our docker images and repositories. We can have our own private registry or use the public docker registry called _Docker Hub_.

## 3. Our first Docker container

We use [Docker Hub](https://hub.docker.com) to find a docker image.

`docker run alpine:latest echo "Hello Nau"` We indicate the name of the image that we want to use (`alpine`), the tag name (`latest`) and the command that we want to execute in this container.

`docker run -i -t alpine:latest sh` We will have a terminal inside this container. `-i` means input, the commands that we are sending. `-t` = tty.

Each time that we use the command `docker run` we are creating a new container (a new instance). For that, if we create some files using the terminal, we will lose them after close ther container.

`docker run -d alpine:latest sleep 10000` If we want to execute our container in the background, we can use the option `-d` (detached).

## 4. Docker commands

`docker images`- Show all the images that we have in our local computer.

`docker ps` - Show all the containers with the state _up_.

`docker ps -a` - Show all the containers.

`docker run --rm alpine:latest sleep 100` - With the `--rm` option, the container will be deleted after its execution.

`docker run --name=nau_container alpine:latest` - With the `--name=container_name` we can give a name to a container.

`docker stop nau_container` - We stop the container.

`docker inspect container_id` - Shows information about the container.

`docker exec -i -t nginx bash` - It permits us to execute a command in a container that is up.

## 5. Mapping ports and share files

`docker run --rm --name nginx -d -p 8080:80 nginx` - With this, we map the port 8080 of our computer to the port 80 of the container.

`docker run --rm -d --name nginx -p 8080:80 -v $(pwd):/usr/share/nginx/html nginx` - With the option `-v` we can mount the directory that we have in our computer (`$(pwd)`) in the directory inside the container (`/usr/share/nginx/html`).

## 6. Docker logs

`docker logs nginx` - We can read the logs of the container called `nginx`.

`docker logs -f nginx` - With the option `-f` (follow) the log will keep open and refreshing similar to `tail -f`.

## 7. Docker images layers

The images in Docker are made up by a list of only-read layers that represent each change in the file system. This layers form a stack to create the base of the container file system. Over these only-read layers, exists a writable content layer that belongs to each container.

## 8. Creating Docker images with Docker commit

<p align="center">
  docker commit &lt;container_id&gt; &lt;image_name&gt;:&lt;image_tag&gt;
</p>

  1. Initiate a conainter using a base image - `docker run -it debian:stretch bash`
  2. We install in this container all that we need (for example, igt) - `apt-get update; apt-get install git`(In bash inside the container)
  3. We make commit after our changes - `docker commit dd141af2ebb4 git:2.11.0`
  4. Now, we can create a new container using this new image - `docker run -it git:2.11.0 bash`

## 9. Creating Docker images with a Dockerfile

This is the best way to create an image because it lets us to make a set of actions that we usually can't with docker commit (for example, create environment variables or set a default command). With it, we atomatize all the processes because a Dockerfile is a text file whose contains all the instructions added by the user to create the image.

Each instruction adds a new layer to the image.

  1. `vim Dockerfile`
  2. Write our Dockerfile:
  
  ```dockerfile
  FROM debian:stretch # Base image
  RUN apt-get update
  RUN apt-get install -y git # Important to add the -y option cause we do not want to be promted to confirm the installation
  RUN apt-get install -y vim
  ```
  3. `docker build -t git:2.11.0 .`
  
<p align="center">
  docker build -t &lt;image_name&gt;:&lt;tag_name&gt; &lt;docker_context_route&gt;
</p>

  · `-t` We use it to give a name and a tag to this image. If we don't give a tag, docker will use the tag 'latest'.

  ### Dockerfile best practices
  
  #### Chain instructions
  
  Each RUN command in our Dockerfile generates a new image that is commited and then, it is removed the before image. For that, is better to chain all the RUN instructions to reduce the number of layers generated. We can do it using '&&':
  
  ```dockerfile
  FROM debian:stretch
  RUN apt-get update && apt-get install -y git && apt-get install -y vim
  ```
    
  #### Sort multiline arguments alphabetically
  
  ```dockerfile
  FROM debian:stretch
  RUN apt-get update && \
      apt-get install -y \
      git \
      nginx \
      vim
  ```

  ### Dockerfile commands
