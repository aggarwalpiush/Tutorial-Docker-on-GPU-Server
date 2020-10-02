Docker provide different platform agnostic and hardware agnostic environments on same machine. For example, if an application need linux operating system to perform its operation, it is feasible on mac/windows or other operating system by creating a docker container. 

In this tutorial, we will focus on installation and creation of docker containers specifically on GPU servers. so, if you are new to docker, more detailed explainations are available at https://docker-curriculum.com/

Basic docker commands can be hands-on in its playground available at https://labs.play-with-docker.com/


### Docker installation on GPU integrated servers

GPU enabled servers need cuda or (cuda + cudnn) drivers which provide parallel computing platform and programming model to develop GPU-accelerated applications. Therefore, it is mandatory to install these drivers first hand.

To check whether these drivers are already installed. Use the following command:

``` 

nvidia-smi 
nvcc --version

```

In case of unavailability of cuda drivers use this [link](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#package-manager-installation) to install them. 


If docker is already installed in your machine, existing version of docker can be identified using following command:

```
docker version
```

If Docker version is less or equal to 19.02 , GPU can be accessible from inside docker’s containers using plugin called ``` nvidia-docker ```. 

To verify whether ``` nvidia-docker ``` plugin is available, test it using latest CUDA image.

```
docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi

```

If plugin is not available, either upgrade docker (**Recommended**) to its latest version or install ``` nvidia-docker ``` plugin.

To upgrade docker to its latest version, use following commands:

```
sudo apt-get update
sudo apt-get --only-upgrade install docker-ce nvidia-docker2
sudo systemctl restart docker
```

If you dont want to update the docker version, To install ``` nvidia-docker ``` plugin, execute following steps:

Step 1: Remove NVIDIA docker 1.0 (if installed)

```
docker volume ls -q -f driver=nvidia-docker | xargs -r -I{} -n1 docker ps -q -a -f volume={} | xargs -r docker rm -f
sudo apt-get purge -y nvidia-docker
```

Step 2: add the necessary repository, then update the apt package index:

```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update
```

Step 3: install ``` nvidia-docker ``` plugin:

```
sudo apt-get install -y nvidia-docker2
sudo pkill -SIGHUP dockerd
```

Step 4: Verify the installation

Lets run latest CUDA image

```
docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi

```

From Docker version [19.03](https://github.com/moby/moby/pull/38828), Docker provide device requests modules to support NVIDIA GPUs. So, installation of docker plugin is not required.

In case of unavailability of Docker, it can be installed by using instructions provided in the link: https://docs.docker.com/engine/install/


### General usage:

#### Test nvidia-smi with the latest official CUDA image
```
docker run --gpus all nvidia/cuda nvidia-smi
```

Note: To use gpu within the docker container, all the images must include cuda drivers. Therefore we have used ```nvidia/cuda``` which is one of the image that include these drivers.

#### Start a GPU enabled container on two GPUs
```
docker run --gpus 2 nvidia/cuda nvidia-smi
```

#### Starting a GPU enabled container on specific GPUs
```
docker run --gpus '"device=1,2"' nvidia/cuda nvidia-smi
docker run --gpus '"device=UUID-ABCDEF,1"' nvidia/cuda nvidia-smi
```


#### Create a GPU-enabled docker container with python tensorflow library in ubuntu distribution.

```
docker run --gpus all -it tensorflow/tensorflow:latest-gpu bash
```

#### Create a GPU-enabled docker container with python pytorch library in ubuntu distribution.

```
docker run --gpus all -it  pytorch/pytorch:latest bash
```

Note: Here ``` tensorflow/tensorflow:latest-gpu ``` and ``` pytorch/pytorch:latest ``` are docker images available at [docker hub](https://hub.docker.com/). To get interactive pseudo tty session ``` -it ``` or ``` -i -t ``` or ``` -ti ``` can be used.

#### To list existing running containers

```
docker container ls
```

Note: In the output, first column will give you <containers id> of your container. 
  
#### To list existing all containers
 
```
docker container ls -a
```

#### To attach a container

Make sure container is running 

```
docker attach <container id>
```

if container is not running, start the container first by executing following command

```
docker start <container id>
```


#### To stop container service

Inside the container session

```
exit
```

Outside the container or in the main server

```
docker stop <container id>
```

#### To rename container

While creating the containers, if container name is not provided, docker name the container with some random string(can be found in the last column of container list). To rename the string with your chosen name following command can be used.

```
docker rename oldname newname
```

#### To remove container

```
docker rm <container id>
```

#### To list images

```
docker image ls
```

#### To remove image

To remove or delete docker image, make sure there is no linked container exist to that image.

```
docker image rm <image id>
```


### Import and Export of containers

To export docker container, it can be saved in zipped file, 

```
docker export <container id> | gzip > filename.gz
```

To import and interactive run docker container available in zipped file format

```
zcat filename.gz | docker import - <container name>
docker run -i -t <container name> /bin/bash
```

### Dockerfiles

Main task of dockerfile to build the Docker images. Building of Docker image represents cloning of development environment (the software application along with host os and installed dependent libraries developed from scratch or on the top of existing docker image) into a portable structure called image, in case an application host os include cuda drivers, then these drivers are also part of this image.  

Dockerfile is created with list of instructions, most common are:

#### FROM

If you are performing When you run an image and generate a container, you add a new writable layer (the “container layer”) on top of the underlying layers. 



### Build Images

TODO

### Open a Jupyter notebook in your local machine while running on docker container which is created in remote gpu/cpu server

Idea: To forward the notebook html from your docker container to remote server to your local machine.

#### Steps:

In your local machine (tested on macOS with common port number 9999 at each place) terminal, run the following command:

```
ssh -L <unused local port number>:localhost:<unused remote server port number> <username>@<remote server ip>
```

Create a docker container in your remote machine with port forwarding information

```
docker run -it --gpus all -p <unused remote server port number>:<unused remote server port number>  <docker image>  bash
```

In your docker container open the jupyter notebook with following command (make sure jupyter notebook is installed in your container)

```
jupyter notebook --ip 0.0.0.0 --port <unused remote server port number> --allow-root
```
You will get the url asked to run into your browser along with token, you will use this token id in next step.

Now, in your local machine, open the browser and run following url:

```
http://localhost:<unused remote server port number>/
```

Enter the token mention in previous step.


### Open a tensorboard in your local machine while running on docker container which is created in remote gpu/cpu server

Similar to above steps, we need to provide tensorboard port which is 6006 by default.


#### Steps:

In your local machine (tested on macOS with common port number 9999 at each place) terminal, run the following command:

```
ssh -L 6006:localhost:<6006 <username>@<remote server ip>
```


Create a docker container in your remote machine with port forwarding information

```
docker run -it --gpus all -p 6006:6006  <docker image>  bash
```

if you want to run tensorboard in the jupyter notebook which is accessed in your local machine but running in your GPU server, you need to pass two ports ids (one for jupyter notebook and other is for your tensorboard)

```
docker run -it --gpus all -p 6006:6006  -p <port for jupyter notebook>:<port for jupyter notebook>  <docker image>  bash
```

In your docker container register the tensorboard command that in python file or in jupyter notebook in your local machine.


```
%tensorboard --bind_all --logdir logs  --port 6006
```

