# MXNet SSD

This helps you get setup for running the train_end2end example described in the
[ssd example for MXNet](https://github.com/dmlc/mxnet/tree/master/example/ssd).

You do not need to have a copy of MXNet's source code in order to run it, though you may download it
and map it your filesystem copy into the container if you wish (see examples below).

In addition to providing an reproducible environment (and producing a docker image that can be shared across
machines), an added benefit is being able to use docker volumes to dynamically set:

- the pretrained network
- the path to the output data that is captured
- the (VOC-like) dataset that is used for training

## Quick start

You need [CUDA 8](https://developer.nvidia.com/cuda-toolkit) and [NVIDIA Docker](https://github.com/NVIDIA/nvidia-docker).

Build the container

```
$ docker build -t mxnet-ssd mxnet-ssd
```

Download pre-trained networks and VOC devkit and 

```
$ mkdir -p ssd-demo/pretrained-networks
$ cd pretrained-networks && wget https://dl.dropboxusercontent.com/u/39265872/ssd_300_vgg16_reduced_voc0712_trainval.zip && unzip ssd_300_vgg16_reduced_voc0712_trainval.zip && cd -
$ mkdir -p ssd-demo/data
$ cd ssd-demo/data && wget http://host.robots.ox.ac.uk/pascal/VOC/voc2012/VOCtrainval_11-May-2012.tar && tar -xf VOCtrainval_11-May-2012.tar && cd -
```

**Train** the network, capturing output to a directory.

```
$ mkdir -p ssd-demo/training-runs/mxnet-ssd-voc/cache
$ nvidia-docker run --rm --name run-mxnet-ssd-train-2017-02-28T15-52-00 \
  -v `pwd`/ssd-demo/training-runs/mxnet-ssd-voc:/media/ngv/output \
  -v `pwd`/ssd-demo/training-runs/mxnet-ssd-voc/cache:/root/mxnet/example/ssd/dataset/cache \
  -v /mnt/ngv/pretrained-networks/mxnet/ssd:/media/ngv/pretrained:ro \
  -v `pwd`/ssd-demo/data/VOCdevkit:/root/mxnet/example/ssd/data/VOCdevkit:ro \
  -it mxnet-ssd \
  python train.py \
  --image-set trainval \
  --year 2012 \
  --val-image-set '' \
  --val-year '' \
  --pretrained /media/ngv/pretrained/vgg16_reduced \
  --prefix /media/ngv/output/ssd \
  --end-epoch 100 \
  --gpus 0 \
  --log '' \
  2>&1 | tee ssd-demo/training-runs/mxnet-ssd-voc/training-logs.txt
```

**Test** the network, using the parameters from the final epoch of training, capturing
the results to a directory of our choosing.

(TODO)


## Details

### What's built into the docker image

Instead of using [an off the shelf MXNet docker image](https://github.com/Kaixhin/dockerfiles/blob/master/cuda-mxnet/cuda_v8.0/Dockerfile)
we build off nvidia's [nvidia/cuda:8.0-cudnn5-devel](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/8.0/devel/cudnn5/Dockerfile)
and customize the build of MXNet to include additional operators required by the SSD example.

We also specify a specific release to make the building of the image reproducible; if you wish a more recent build,
update or remove the git checkout command in the dockerfile, test it out, and then make a change to this dockerfile.

Finally, we install the required additional apt and pip dependencies, and build the RCNN dependencies by running
`make` within the directory.

### Running the container

#### Capturing the output of the run

You need to use [docker volumes](https://docs.docker.com/engine/tutorials/dockervolumes/)
map a directory into `/media/ngv/output` in order capture the output, including the trained network params for
each epoch and logs from the run. See example below.

#### Training against other datasets

This container can run against any dataset in VOC format by using a docker volume, just map your devkit directory
(the one containing VOC2012) into `/root/mxnet/example/rcnn/data/VOCdevkit`.

See example below.

## Examples

Unlike the quick start, the following examples assume that the pretrained vgg16 network and VOC dataset are located
under `/mnt/ngv`.

### Building the container

First you need to build the container:

```
$ docker build -t mxnet-ssd mxnet-ssd
```

### Capturing output from training run against vanilla VOC

In this example, we create a directory to capture the output, and then map its path to `/media/ngv/output`
in order to capture the logs and network parameters from training. VOC is used for training by default.

TODO

### Capturing output from training run against our own dataset in voc format

Same as previous example, but we also map in kitti in voc format from our local file system
to `/root/mxnet/example/ssd/data/VOCdevkit`.

TODO

### Mapping in MXNet source code

In this example, we shadow the MXNet code with a copy from our filesystem. This can be handy if you wish
to alter / iterate on the code.

If necessary, clone MXNet (and make any modifications you'd like):

```
$ git clone git@github.com:dmlc/mxnet.git
```

Then map a subdirectory into the container to shadow `/root/mxnet/example/ssd/dataset`.

TODO