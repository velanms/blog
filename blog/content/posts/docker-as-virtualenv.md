---
author: "Velan Salis"
title: "Docker, a replacement of Virtualenv for Python developement"
date: "2018-01-10"
description: "A post about using docker to improve the workflow related to python development"
categories: ["Tutorial"]
tags: ["docker", "python", "virtualenv"]
slug: "docker-as-virtualenv"
---

Docker! Sounds fancy and simple but trust me it has revolutionized how the tech industry builds, ships and deploys the applications lately. **Docker** is basically a mini-operating system where you build and run your applications which is totally isolated from the native operating system in your computer, and the same container can be deployed in the hosting platforms. If you are new to docker and want to learn more about its intricacies, i highly encourage you to go through the links given in the end of this article to learn more about it. So let’s get right in :)

### Creating a Dockerfile

Dockerfile is the recipe of an image. Dockerfile specifies how your image should look like. However, we are not creating an image from the scratch. We will take the existing image and add our dependencies on top of that to create a customized image based on our requirements. You can setup the Dockerfile to either take the existing requirements.txt file and install dependencies from pip or install dependencies manually without requirements.txt. This is how the Dockerfile will look like in either of the situations.

```
    FROM python:3

    RUN mkdir /usr/src/app
    WORKDIR /usr/src/app

    # Below two steps are valid only if you have
    # requirements.txt in the project folder
    COPY requirements.txt .
    RUN pip install -r requirements.txt

    # If there is no requirements.txt,
    # You can install individual packages as follows
    # RUN pip install <package-1> <package-2>...<package-n>
```
- FROM: The image that we are building has to be built in such a way that we take an existing image and add our own functionalities to it. So we are choosing an image with python 3.x since we are dealing with Python developement. However, You can choose any image or any version you want.

- RUN: This will run a command inside the image. Since our image is a mini linux OS (usually debian distribution) we can run linux commands inside it. Here we run mkdir /usr/src/app which creates a working directory for our application to run. Which is not absolutely necessary since its automatically created in the next step. This step was a demonstration of how to run commands inside an image.

- WORKDIR: This specifies the working directory of our image. After setting the WORKDIR, Any RUN, CMD, ENTRYPOINT, COPY and ADD commands we execute, will be executed inside the WORKDIR . In our case, its /usr/src/app .

- COPY: This command will copy any file specified on the left before the space to the directory specified on the right of the space. In our case, we are copying requirements.txt from our project folder on the host to our WORKDIR /usr/src/app in the image And running pip install -r requirements.txt on /usr/src/app which will install and save all the pip dependencies inside the image.

If there is no requirements.txt, you can skip the COPY step and install pip dependencies by specifying the pip install <package name> command in the RUN section one after the other. And save the Dockerfile with the name Dockerfile

We successfully created a recipe of how our image should look like, now the next step is to build the image based on this recipe.

### Building from the Dockerfile

Now since we are done with creating our recipe, we need to build our own image from the recipe. To build the image, the command is as follows

    docker build -t python/pack .

**docker build** is the command that tells docker that we need to build our own image. **-t python/pack** is where we add our own name for the new docker image. we can also add tags to the image using -t python/pack:latest where latest is the tag for the image. (although latest is a default tag for any new image) **.** tells the docker to use the current directory in lookout for Dockerfile . If the Dockerfile lies in different directory, you can specify the path from where you would want to build the Dockerfile . Once the build is finished, you will have a Customized docker image based on your recipe, and you can check it by running docker images command in the shell. You should see an image called python/pack

### Creating a Container

Containers are the place where we execute our programs by mounting our local volumes to the container volumes. From where you can access the packages and dependencies installed in the container meanwhile, persisting the data to the host system. Run the following command in bash

    docker run --rm -it \
    -u $(id -u):$(id -g) \
    -v /etc/passwd:/etc/passwd \
    -v $(pwd):/usr/src/app \
    python/pack \
    bash

This creates a container based on the image python/pack where you can accesss your project files from the directory /usr/src/app and then you can execute the code as you would do in your host machine. Lets break this command down line-by-line:

- docker run --rm -it : docker run tells the docker daemon that we want to create a new container, **-it **is where we run this container in the interactive mode. -i basically keeps the STDIN open and -t allocates a pseudo TTY. In simple words, -it is essential to open a bashshell inside the container, later. You can also use **-d** instead of **-it** which starts the container in detached mode, which starts the container in the background.

- -u $(id -u):$(id -g) : It is not recommended to start a container as a root user. So we specify -u flag and then user ID $(id -u) and group ID $(id -g) to start a container as a regular user. Now we will have previliges over only that directory which is mounted to the container and not the entire container. This is the cleaner way of creating a container.

- -v /etc/passwd:/etc/passwd: This -v argument mounts /etc/passwd of the host to /etc/passwd of the container. Otherwise the docker container can’t access the usernames and display I have no name! in the bash prompt. Mounting /etc/passwd solves this problem.

- -v \$(pwd):/usr/src/app: This -v argument mounts your project directory to /usr/src/app by doing which you can access your code in the project directory inside the docker container and the changes made inside the docker container will be available in the project directory (pwd prints the present working directory). So in simple words, this creates a shared space between the container and the host machine from where the host machine can access the packages and dependencies installed in the docker image.

- python/pack: This is pretty straight-forward. This is where we mention, which image is the container will be based on. The resulting container will contain the architecture of the mentioned image.

- bash: This is where we mention the command to be executed soon after the container is created. Since we want the bash prompt, we specify the command bash which opens a bash shell soon after the container is created.

By the end of this process, we will have a container which is built with the dependencies we need, sharing a same space as our project directory, totally isolated from out host machine. This is really useful since the dependencies are not installed in the host machine. The host machine only contains the project folder but it runs inside the customized container. Clean stuf !!

### Extras/Troubleshooting

- **Editing a Docker Image :** Let’s say you have created a Docker image with 3 packages and now you want to install a new package to the already built image. You can do it by editing the imageDockerfile and running the same command docker build -t python/pack . and docker will make changes to the existing image from where the content of the Dockerfile has changed. To make these changes, it is important to note that Dockerfile is in the same folder as it was initially built. Because Docker daemon uses current Dockerfile folder as a build context and when changes to the Dockerfile is made, build context will be checked and from there the Image will be built.

- **Port Forwarding :** Port forwarding is such a useful functionality inside docker container. Let’s say you are using Jupyter notebook inside your docker container and it exposes a port 8888. But it will be on the context of the container, and not on the host machine. So if you go to localhost:8888 in the host machine, it won’t work. So you map the host’s 8888 port to container’s 8888 port when creating the container and then you can access jupyter notebook by going to localhost:8888 on the host machine. You can map a port by adding -p 8888:8888 when creating the container where left side of the : is the port on the host and the right side is the port on the container.

### Conclusion

Docker is a revolutionary tech, there is no doubt about that. Once the developement is over, the developed project can be containerized and the container can be deployed in hosting platforms like Digital Ocean. Containers can be made to talk to each other and each container can be used as a microservice. The possibilities are endless. It depends on you how would use this tech to the fullest. Happy coding :)

### Essential Docker Commands

- Pull Image from Dockerhub : `docker pull <image name>`
- Build a Docker Image : `docker build -t <image name> <path>`
- Create Docker Container :
  `docker run [-it/-d] --rm -u <user id>:<group id> -v <host directory>:<container directory> -p <host port>:<container port> <image name> <initial command>`
- List Running Container : `docker ps`
- List All Containers : `docker ps -a`
- List All Container ID's : `docker ps -aq`
- Delete Single Container : `docker rm <container id>`
- Delete All Docker Containers : `docker rm $(docker ps -aq)`
- List all Images : `docker images`
- List all Image ID's : `docker images -aq`
- Delete single image : `docker rmi <image id>`
- Delete all images : `docker rmi $(docker images -aq)`
- Start Interactive session with a container started in detached mode (-d) : `docker exec -it <container_name> bash`

### Important Links

- [Installing Docker](https://docs.docker.com/install/)

- [Post Installation Steps](https://docs.docker.com/install/linux/linux-postinstall/)

- [Dockerfile, full reference](https://docs.docker.com/engine/reference/builder/)

- [Compose file reference](https://docs.docker.com/compose/compose-file/)