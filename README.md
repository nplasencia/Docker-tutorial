<p align="center">
  <img src="https://cdn.worldvectorlogo.com/logos/docker.svg" width="250px"/>
</p>

# Docker

It will help us with the automation of our deploys using _containers_. This adds control and security to our deploys.

## 1. Installation

[Linux](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-engine---community) - [Mac](https://hub.docker.com/editions/community/docker-ce-desktop-mac)

## 2. Concepts

### - Images:

Read only templates used to create containers. They are created using the commands `docker_build` or `docker_commit`.

The images are created using other images.

They are saved in a _docker registry_.

### - Containers:

If we think in an OOP class, the __container__ is the instance of an image. The containers are a light and portable encapsulation of an environment where our applications are executed.

### - Repositories:

It is a collection of different images with the same name but different tag. Usually, each tag represents a different version of the image.

### - Registries:

A __registry__ is the place where we have stored our docker images and repositories. We can have our own private registry or used the public docker registry called _Docker Hub_.
