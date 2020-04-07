Docker containerization help to create different platform agnostic and hardware agnostic environments on same machine. For example, if an application need linux operating system to perform its operation, it is feasible on mac/window or other operating system by making docker container. 

In this tutorial, we will focus on installation and creation of docker container specifically on GPU servers. so, if you are new to docker, more detailed explainations are available at https://docker-curriculum.com/

Basic docker commands can also be played in its playground which is available at https://labs.play-with-docker.com/



### GPU + CPU server

GPU enabled servers need cuda or (cuda + cudnn) drivers which provide parallel computing platform and programming model to develop GPU-accelerated applications. 

If docker is already installed in your machine, existing version of docker can be identified using following command:

```
docker version
```

If Docker version is less or equal to 19.02 , GPU can be accessible from inside docker’s containers using plugin called ``` nvidia-docker ```. 

To install ``` nvidia-docker ``` plugin, 

Step 1: Remove NVIDIA docker 1.0 (if installed)

```
docker volume ls -q -f driver=nvidia-docker | xargs -r -I{} -n1 docker ps -q -a -f volume={} | xargs -r docker rm -f
sudo apt-get purge -y nvidia-docker
```

Step 2: add the necessary repository, then update the apt package index:

```
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/ubuntu16.04/amd64/nvidia-docker.list | \
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

From Docker version 19.03

```
#### Test nvidia-smi with the latest official CUDA image
docker run --gpus all nvidia/cuda:10.0-base nvidia-smi

# Start a GPU enabled container on two GPUs
docker run --gpus 2 nvidia/cuda:10.0-base nvidia-smi

# Starting a GPU enabled container on specific GPUs
docker run --gpus '"device=1,2"' nvidia/cuda:10.0-base nvidia-smi
docker run --gpus '"device=UUID-ABCDEF,1"' nvidia/cuda:10.0-base nvidia-smi

# Specifying a capability (graphics, compute, ...) for my container
# Note this is rarely if ever used this way
docker run --gpus all,capabilities=utility nvidia/cuda:10.0-base nvidia-smi
```





