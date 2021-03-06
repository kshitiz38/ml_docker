## Docker image for Deep Learning
The Dockerfiles with cpu and gpu build contains Tensorflow, Caffe, Keras and Theano libraries along with OpenCV and popular python libraries. The CPU version should work on Linux, Windows and OS X. The GPU version will, however, only work on Linux machines. See [OS support](#what-operating-systems-are-supported) for details

## Specs
This is what you get out of the box when you create a container with the provided image/Dockerfile:
* Ubuntu 16.04
* [CUDA 8.0](https://developer.nvidia.com/cuda-toolkit) (GPU version only)
* [cuDNN v6.0.21](https://developer.nvidia.com/cudnn) (GPU version only)
* [Tensorflow 1.1.0](https://www.tensorflow.org/)
* [Caffe](http://caffe.berkeleyvision.org/)
* [Theano 0.90](http://deeplearning.net/software/theano/)
* [Keras 2.0.5](http://keras.io/)
* [iPython/Jupyter Notebook](http://jupyter.org/) (including iTorch kernel)
* [Numpy](http://www.numpy.org/), [SciPy](https://www.scipy.org/), [Pandas](http://pandas.pydata.org/), [Scikit Learn](http://scikit-learn.org/), [Matplotlib](http://matplotlib.org/)
* [OpenCV](http://opencv.org/)
* A few common libraries used for deep learning

## Setup
### Prerequisites
1. Install Docker following the installation guide for your platform: [https://docs.docker.com/engine/installation/](https://docs.docker.com/engine/installation/)

2. **GPU Version Only**: Install Nvidia drivers on your machine either from [Nvidia](http://www.nvidia.com/Download/index.aspx?lang=en-us) directly or follow the instructions [here](https://github.com/saiprashanths/dl-setup#nvidia-drivers). Note that you _don't_ have to install CUDA or cuDNN. These are included in the Docker container.

3. **GPU Version Only**: Install nvidia-docker: [https://github.com/NVIDIA/nvidia-docker](https://github.com/NVIDIA/nvidia-docker), following the instructions [here](https://github.com/NVIDIA/nvidia-docker/wiki/Installation). This will install a replacement for the docker CLI. It takes care of setting up the Nvidia host driver environment inside the Docker containers and a few other things.

### Obtaining the Docker image
You have 2 options to obtain the Docker image
#### Option 1: Download the Docker image from Docker Hub
Docker Hub is a cloud based repository of pre-built images. You can download the image directly from here, which should be _much faster_ than building it locally (a few minutes, based on your internet speed). Here is the automated build page for `ml-docker`: [https://hub.docker.com/r/zitihsk/ml-docker/](https://hub.docker.com/r/zitihsk/ml-docker/). The image is automatically built based on the `Dockerfile` in the Github repo.

**CPU Version**
```bash
docker pull zitihsk/ml-docker:cpu
```

**GPU Version**
```bash
docker pull zitihsk/ml-docker:gpu
```
The automated build for gpu does not have opencv3 installed due to build timeout from docker however you can run command listed in /root/opencv_readme.txt or edit it in Dockerfile and build locally using option described below

#### Option 2: Build the Docker image locally
Alternatively, you can build the images locally. Also, since the GPU version is not available in Docker Hub at the moment, you'll have to follow this if you want to GPU version. Note that this will take an hour or two depending on your machine since it compiles a few libraries from scratch.

```bash
git clone <…>
cd ml-docker
```

**CPU Version**
```bash
cd cpu
docker build -t  zitihsk/ml-docker:cpu -f Dockerfile .
```

**GPU Version**
```bash
cd gpu
docker build -t  zitihsk/ml-docker:gpu -f Dockerfile .
```
This will build a Docker image named `ml-docker` and tagged either `cpu` or `gpu` depending on the tag your specify. Also note that the appropriate `Dockerfile` has to be used.

## Running the Docker image as a Container
Once we've built the image, we have all the frameworks we need installed in it. We can now spin up one or more containers using this image.

**CPU Version**
```bash
docker run -it -p 8888:8888 -p 6006:6006 -p 5000:5000 -v /sharedfolder:/root/sharedfolder  zitihsk/ml-docker:cpu bash
```

**GPU Version**
```bash
nvidia-docker run -it -p 8888:8888 -p 6006:6006 -p 5000:5000 -v /sharedfolder:/root/sharedfolder  zitihsk/ml-docker:gpu bash
```
Note the use of `nvidia-docker` rather than just `docker`

| Parameter      | Explanation |
|----------------|-------------|
|`-it`             | This creates an interactive terminal you can use to iteract with your container |
|`-p 8888:8888 -p 6006:6006 -p 5000:5000`    | This exposes the ports inside the container so they can be accessed from the host. The format is `-p <host-port>:<container-port>`. The default iPython Notebook runs on port 8888 and Tensorboard on 6006 and for flask server is 5000|
|`-v /sharedfolder:/root/sharedfolder/` | This shares the folder `/sharedfolder` on your host machine to `/root/sharedfolder/` inside your container. Any data written to this folder by the container will be persistent. You can modify this to anything of the format `-v /local/shared/folder:/shared/folder/in/container/`. See [Docker container persistence](#docker-container-persistence)
|`zitihsk/ml-docker:cpu`   | This the image that you want to run. The format is `image:tag`. In our case, we use the image `ml-docker` and tag `gpu` or `cpu` to spin up the appropriate image |
|`bash`       | This provides the default command when the container is started. Even if this was not provided, bash is the default command and just starts a Bash session. You can modify this to be whatever you'd like to be executed when your container starts. For example, you can execute `docker run -it -p 8888:8888 -p 6006:6006 -p 5000:5000 zitihsk/ml-docker:cpu jupyter notebook`. This will execute the command `jupyter notebook` and starts your Jupyter Notebook for you when the container starts

### Data Sharing
See [Docker container persistence](#docker-container-persistence).
Consider this: You have a script that you've written on your host machine. You want to run this in the container and get the output data (say, a trained model) back into your host. The way to do this is using a [Shared Volume](#docker-container-persistence). By passing in the `-v /sharedfolder/:/root/sharedfolder` to the CLI, we are sharing the folder between the host and the container, with persistence. You could copy your script into `/sharedfolder` folder on the host, execute your script from inside the container (located at `/root/sharedfolder`) and write the results data back to the same folder. This data will be accessible even after you kill the container.

## What is Docker?
[Docker](https://www.docker.com/what-docker) itself has a great answer to this question.

Docker is based on the idea that one can package code along with its dependencies into a self-contained unit. In this case, we start with a base Ubuntu 14.04 image, a bare minimum OS. When we build our initial Docker image using `docker build`, we install all the deep learning frameworks and its dependencies on the base, as defined by the `Dockerfile`. This gives us an image which has all the packages we need installed in it. We can now spin up as many instances of this image as we like, using the `docker run` command. Each instance is called a _container_. Each of these containers can be thought of as a fully functional and isolated OS with all the deep learning libraries installed in it.

## Why do I need a Docker?
Installing all the deep learning frameworks to coexist and function correctly is an exercise in dependency hell. Unfortunately, given the current state of DL development and research, it is almost impossible to rely on just one framework. This Docker is intended to provide a solution for this use case.

If you would rather install all the frameworks yourself manually, take a look at this guide: [Setting up a deep learning machine from scratch](https://github.com/saiprashanths/dl-setup)

### Do I really need an all-in-one container?
No. The provided all-in-one solution is useful if you have dependencies on multiple frameworks (say, load a pre-trained Caffe model, finetune it, convert it to Tensorflow and continue developing there) or if you just want to play around with the various frameworks.

The Docker philosophy is to build a container for each logical task/framework. If we followed this, we should have one container for each of the deep learning frameworks. This minimizes clashes between frameworks and is easier to maintain as things evolve. In fact, if you only intend to use one of the frameworks, or at least use only one framework at a time, follow this approach. You can find Dockerfiles for individual frameworks here:
* [Tensorflow Docker](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/tools/docker)
* [Caffe Docker](https://github.com/BVLC/caffe/tree/master/docker)
* [Theano Docker](https://github.com/Kaixhin/dockerfiles/tree/master/cuda-theano)
* [Keras Docker](https://github.com/Kaixhin/dockerfiles/tree/master/cuda-keras)
* [Lasagne Docker](https://github.com/Kaixhin/dockerfiles/tree/master/cuda-lasagne)
* [Torch Docker](https://github.com/Kaixhin/dockerfiles/tree/master/cuda-torch)

## FAQs
### Performance
Running the DL frameworks as Docker containers should have no performance impact during runtime. Spinning up a Docker container itself is very fast and should take only a couple of seconds or less

### Docker container persistence
1. **Commit**: If you make changes to the image itself (say, install a few new libraries), you can commit the changes and settings into a new image. Note that this will create a new image, which will take a few GBs space on your disk. In your next session, you can create a container from this new image. For details on commit, see [Docker's documentaion](https://docs.docker.com/engine/reference/commandline/commit/).

2. **Shared volume**: If you don't make changes to the image itself, but only create data (say, train a new Caffe model), then commiting the image each time is an overkill. In this case, it is easier to persist the data changes to a folder on your host OS using shared volumes. Simple put, the way this works is you share a folder from your host into the container. Any changes made to the contents of this folder from inside the container will persist, even after the container is killed. For more details, see Docker's docs on [Managing data in containers](https://docs.docker.com/engine/userguide/containers/dockervolumes/)

### How do I update/install new libraries?
You can do one of:

1. Modify the `Dockerfile` directly to install new or update your existing libraries. You will need to do a `docker build` after you do this.

2. You can log in to a container and install the frameworks interactively using the terminal. After you've made sure everything looks good, you can commit the new contains and store it as an image

### What operating systems are supported?
Docker is supported on all the OSes mentioned here: [Install Docker Engine](https://docs.docker.com/engine/installation/) (i.e. different flavors of Linux, Windows and OS X). The CPU version will run on all the above operating systems. However, the GPU version will only run on Linux OS. This is because Docker runs inside a virtual machine on Windows and OS X. Virtual machines don't have direct access to the GPU on the host. Unless PCI passthrough is implemented for these hosts, GPU support isn't available on non-Linux OSes at the moment.
