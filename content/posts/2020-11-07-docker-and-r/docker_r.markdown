---
title: "Why Docker & R?"
author: "Alberto Almuiña"
date: '2020-11-07T21:13:14-05:00'
description: Create a Docker container to encapsulate your R processes.
slug: docker_r
tags:
- containers
- Docker
categories:
- Deployment
- Production
---

## What is Docker?

Docker is an open source project that automates the deployment of applications within software containers, providing an additional layer of abstraction and automation of application virtualization across multiple operating systems.

The main difference with a virtual machine is that the containers are much lighter and have a much more efficient consumption of the available resources.

## Why R & Docker?

Docker is a great tool for models and applications deployment. Also avoids major dependency conflicts. For example, imagine the following situation: you are creating a model with R 3.5 and library 'A' 1.0 and you need to share your model with a partner. Your partner has R 3.6 and library 'A' 2.0 installed and if he updates his version to that of your model, other projects break. This is where docker comes into play.

### What's the difference between Docker Image and Container?

Images are those that are built from a set of instructions and indicate the configuration of the operating system (This is where you install all the necessary dependencies and packages and the versions of the programs you need).
Containers are instances of that image running. What does this mean? You create your image once with the necessary settings and each time you launch it, a container will be created.

## Dockerfile and Build an Image

Docker Images are built from a file called Dockerfile. This file indicates the image from which our image will be created, how to configure the operating system, the necessary programs / packages and what the container will do when it is launched.

We are going to create an image in Docker that will do the following (In this example, we created the image in the Debian 10 OS):

1. Install the required R version.

2. Install magick and DSpoty packages (with its corresponding version).

3. Set an environment parameter (will be an artist name, 'Arctic Monkeys' by default).

4. Use the Dspoty library to get a link to the artist photo passed by parameter and save it with the magick library inside the container.

5. Export this image from the container to our host.


Luckily, **rocker** has uploaded several images that have R installed to the DockerHub repository, so this will be our starting image. In this case, we are going to choose an image that has R 3.6.3 installed as a starting point.
This is what is specified in the first line of the Dockerfile (it must always start with a 'FROM' indicating the image from which it starts). 

Second, we define an environment variable. This variable will be the name of the artist from whom we want to obtain a photo. Every time we run an instance of the docker image (that is, we create a container), we must specify this value if we want it to be different from the default value.

Subsequently, we have three RUN commands (these commands are only run once when the image is built). In this case, a directory is created inside the image to work on it and the necessary packages are installed to be able to install the three subsequent R libraries.

**Note:** The 'remotes' library allows us to install the desired versions of the rest of R's libraries. We must be aware that if for some reason this package were deleted from the CRAN repository, all of our remaining libraries could not be installed and would fail. (This also applies to other libraries, but unless they are rare libraries, we should have no problems). In any case, if they are of vital importance to our business / task, a good option would be to have our own central repository with all the libraries we need so that they are always there.


The last two steps are the copy of our R script inside the image and finally we use the CMD instruction (this will be launched every time we run a container of the image) in which the launch of the R script is specified and we move the image to another directory.

This is the Dockerfile used:

![Dockerfile](/img/docker_r_files/docker_r_dockerfile.jpg)

Here is the R script used:

![R Script](/img/docker_r_files/rscript.jpg)

Finally, you need to build the image with the 'build' command and then run the container with the 'run' command.
Note that the -v option is used to specify the equivalence between the host folder and the container one, so that the image saved inside the container is not lost:

![Docker Commands](/img/docker_r_files/commands.jpg)


You can find the images in the expected folder:

![Results](/img/docker_r_files/results.jpg)


**Enjoy!**
