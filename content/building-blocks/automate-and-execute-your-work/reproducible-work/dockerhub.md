---
title: "Export a Python environment with Docker and share it through Docker Hub" 
description: "Learn how to export a containerized python environment with Docker, and make it available to your collaborators in Docker Hub "
keywords: "Docker, environment, Python, Jupyter notebook, Docker-Hub "
weight: 2
author: Diego Sanchez Perez
authorlink: "https://www.linkedin.com/in/diego-s%C3%A1nchez-p%C3%A9rez-0097551b8/"
draft: false
date: 2022-02-02T17:05:00+01:00
aliases: 
  - /export/python-environment-with-docker


---

# Export a Python environment with Docker and share it through Docker Hub

Avoid headaches regarding compatibility and installations by sharing your project's environment as a Docker image that can be easily imported by your peers and collaborators from Docker Hub. In this building block, you will learn how to build a Docker image for your python environment, push it to Docker Hub to make it available to other users, as well as the necessary steps to re-import it and run jupyter notebook on it.

{{% tip %}}

If you haven't installed docker on your computer yet, you can see how to do it on the building block on how to [set up Docker](https://tilburgsciencehub.com/building-blocks/configure-your-computer/automation-and-workflows/docker/).

In case you are new to Docker it is recommended that you review some of the basics of Docker before going ahead with this building block. You can do that by visiting TSH's building block on [Docker for reproducible research](https://tilburgsciencehub.com/building-blocks/automate-and-execute-your-work/reproducible-work/docker/), there you will also learn more on how to create your Dockerfiles and build images from them and more!

Also, you can check how to [import an environment's Docker image and run it on a Google Cloud virtual machine](https://tilburgsciencehub.com/building-blocks/automate-and-execute-your-work/reproducible-work/google_cloud_docker/) on our building block on the topic. There you will see the entire process from installing Docker on your Google Cloud VM instance to having your imported environment running on Jupyter notebook within it!

{{% /tip %}}

### Your environment's dockerfile

The starting point for any docker image is a dockerfile, which in this case will contain the instructions to set up in place all the dependencies associated with your environment.

{{% codeblock %}}

Example dockerfile

```dockerfile
FROM continuumio/miniconda3
RUN conda install jupyter
RUN pip install pytest
RUN mkdir /your-env
```

{{% /codeblock %}}

Above you can see an example of a simple dockerfile suited for this building block. As with any dockerfile, it starts with the `FROM` instruction, which indicates which [parent image](https://docs.docker.com/build/building/base-images/) is going to be used. In this case, the parent image is a popular one that comes with [Miniconda](https://docs.conda.io/en/latest/miniconda.html) pre-installed, a minimalist version of the python distribution platform for data science [Anaconda](https://www.anaconda.com/). One of the main advantages of employing this parent image is that it will already contain the elemental components to run python on it without including too many additional elements which may not be of interest for a particular project and may make the container unnecessarily heavy.  

After that the `RUN` instruction, which is used to execute commands within the container, is employed to install jupyter notebook as well as any packages that are necessary to replicate your environment. In this particular example, the `pytest` package is the only one that is going to be downloaded. Note that miniconda includes already some of the most widely used python packages such as Numpy and Pandas.

{{% tip %}}

Alternatively, you could use a Conda `environment.yml` file and the `conda create` command instead of manually specifying all required dependencies as done in the third line of the sample dockerfile. However, bear in mind that the details of this alternative approach are not explored in this building block. You can read more about it [here](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-from-an-environment-yml-file).

{{% /tip %}}

Lastly, the `RUN` instruction is employed again to create a new directory within the image's container that will be used to connect the container filesystem with your pc filesystem. Don't worry if you don't fully grasp this now, you will come back at this later in the building block. 

Feel free to use this example dockerfile as a template to export your environment if it adjusts to your needs! For that, you should add to it the name of the packages included in your environment (instead of the included pytest) and save it as a text file without an extension named "dockerfile" (i.e. the file should be named "dockerfile" not "dockerfile.txt" or anything else, otherwise docker will not recognize it.) 

{{% codeblock %}}

Example dockerfile - Template

```dockerfile
FROM continuumio/miniconda3
RUN conda install jupyter
RUN pip install <your-packages>
RUN mkdir /your-env
```

{{% /codeblock %}}

{{% tip %}}
If your environment requires you to install a specific version of a package you can use `pip install <package>==<version>` after the `RUN` instruction in the dockerfile instead of the regular `pip install <package>` (e.g. `RUN pip install pytest==7.0.1`)
{{% /tip %}}

Once your dockerfile is ready, open the command line in the directory where it is located and run:

`docker build -t <your_image_name> .`

{{% warning %}}

Docker builds images from a dockerfile and its context (i.e. files located alongside the dockerfile), thus it is generally recommended to place your dockerfile in a directory dedicated to storing the dockerfile itself and its context to ensure the building process is run smoothly. If you are building an image for an environment where the dockerfile will mostly cover the installation of the right packages and no contextual actions are taking place you could just place it in an empty directory, however, bear in mind this may not always be the case.

{{% /warning %}}

After having done this, Docker will have generated an image (i.e. a model of your environment) from it, which is now ready to be exported! 

This image will be identified by the name you decided to give to it after the flag `-t` as well as by an alphanumerical image ID of 12 characters assigned automatically by docker.

{{% tip %}}

You can see a list of all your images and the details about them by running `docker images` in your command line.

{{% /tip %}}


### Pushing your image to Docker Hub

[Docker Hub](https://hub.docker.com/) is the most popular platform for hosting and distributing docker images, which are organized in repositories. One of the advantages of hosting your container image on Docker Hub is that anyone willing to use it (i.e. replicate your environment in this case) will be able to do so remotely simply by pulling from their command line.

{{% tip %}}
Before you try to push your image make sure you are properly logged in to your Docker Hub account. To do that run `docker login` in the command line and then introduce your Docker Hub username and password.
{{% /tip %}}

A Docker Hub repository is designed to contain one or multiple versions of a docker image, which are differentiated by their tag. Optionally, it is recommended that you assign an appropriate tag to each of your images within a repository so you can appropriately differentiate them.  However, you don't have to worry if you don't do it, in that case Docker will simply automatically assign the `latest` tag to the last build of your image. To assign a tag to your recently built image you should type the following in the command line:

`docker tag <image_id> <your_image_name>:<tag>`

{{% tip %}}

Remember that image ID is the unique 12-character string associated with each image. Do not confuse it with the name of your image.

Also bear in mind that the default name of your repository will be your Docker Hub username plus "/" and the name you assigned to the image. (i.e. repository name: <your_username>/<image_name>) 

{{% /tip %}}

Once you have assigned a tag to your image, it is time to push it to Docker Hub using the [docker push](https://docs.docker.com/engine/reference/commandline/push/
) command by typing:

`docker push <your_image_name>:<tag>`

Et voilà! Your image is now on Docker Hub ready to be employed by other users. 

### Docker pull

At this point when your environment's image is already available on Docker Hub, getting it will be as simple as using the `docker pull` command to tell docker that you want to import a given image from a docker hub repository. The resulting command line sentence to run will be:

`docker pull <repository_name>:<tag>`

{{% example %}}

Imagine a fictional repository created by the fictional user "mickey_docker" for hosting the different versions of the image "mickeys_environment". The first version of the image has been recently built and now that it will be pushed to Docker Hub, this user will assign an appropriate tag to it (e.g. "1.0") by running:

`docker tag 123456789abc mickey_docker/mickeys_environment:1.0`

Then, to push it, this user will execute the following in the command line:

`docker push mickey_docker/mickeys_environment:1.0`

Finally, when one a collaborator of this user needs to import the environment's image, by typing:

`docker pull mickey_docker/mickeys_environment:1.0`

Docker will pull the image and have it ready to be run in a container locally.

{{% /example %}}

### Executing Jupyter in your environment

Having pulled your image from Docker Hub, it is time to see how to launch jupyter within your environment's container and work in it. For this you will employ a docker-compose file, which gives docker the instructions on how to provide certain services that a container or group of containers may need to work properly. In this case, those services will be the launching of jupyter notebook, and the mounting of a docker [volume](https://docs.docker.com/storage/volumes/), which is a mechanism to connect a container's filesystem with that of your computer. This will be very useful for cases when you need to move files from and to your environment's container. Below you can see an example docker-compose file which is in charge of these services. Also, If you want to learn more about docker-compose files you can check the [official documentation](https://docs.docker.com/compose/).

{{% codeblock %}}

Example docker-compose

```docker-compose
version: "3.8"
services:
  notebook:
    image: <repo_name>:<tag> 
    command: jupyter notebook --port <port_number> --NotebookApp.open_browser=False --NotebookApp.ip='0.0.0.0' --allow-root --NotebookApp.notebook_dir=/your-env
    ports:
      - <port_number>:<port_number>
    tty: true
    restart: always
    volumes:
      - type: bind
        source: ~/your-env
        target: /your-env
```

{{% /codeblock %}}

For the sake of this building block, if you introduce the appropriate repository name, the tag of the image you are interested in, and the port where you want jupyter to be executed (e.g. 4444) this docker-compose will be ready to allow you to work with your imported environment. Again, feel free to copy it and use it as your docker-compose template! It has been designed to work properly in conjunction with the dockerfile above by following the steps within this building block. For that you should save it (for example, in the same directory where you placed your dockerfile previously) as "docker-compose.yml", open the command line there and run:

`docker compose up`

Now you just have to access jupyter by clicking on the link generated in your command line, or by copying it into your browser's search bar.

<p align = "center">
<img src = "../img/command_line_jupyter.png" width="750">
<figcaption> Click on the generated link or paste it into your browser to access jupyter. </figcaption>
</p>

If the execution was successful, once you access jupyter you will notice that it has been opened in a directory called "your-env". This directory exists in both your local computer and the image's container and is the connection between the two. This is the main function of the [docker volume](https://docs.docker.com/storage/volumes/) that was set in the docker-compose and it implies that any file you place on this directory on your pc will be made available also for you to work with it in your environment's container. Conversely, you can retrieve from the container to your local filesystem any file generated which is stored here. Ideally, here is where you should place the files of your project so you can work with them more flexibly.

You can also use any name for this directory where the volume has been mounted (first in the dockerfile and then in the docker-compose) to make it more recognizable to you.

<p align = "center">
<img src = "../img/dockerfile-your-env.png" width="400">
</p>

<p align = "center">
<img src = "../img/docker-compose-your-env.png" width="500">
<figcaption> Change the name of the directory where the volume is mounted to whichever you prefer. Just make sure that it is the same across both the dockerfile and docker-compose files. </figcaption>
</p>

### Creating new versions of your images with "docker commit"

Imagine that during your project, you have started to use a new python package not included in the original image that you already made available to other users on Docker Hub. In that case, if you have this package already installed in your image's container, you could simply use the [docker commit](https://docs.docker.com/engine/reference/commandline/commit/) command to create a new version of your image including these changes and then push it. For that you just have to run:

`docker commit <container_id> <repo_name>:<new_tag>`

As a result, Docker will have created a new image that contains all the newly installed packages which you can push to your repository as previously shown. 


