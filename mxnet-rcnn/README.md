# MXNet RCNN end2end

This helps you get setup for running the train_end2end example described in the
[rcnn example for MXNet](https://github.com/dmlc/mxnet/tree/master/example/rcnn) as of 1/22/2017.

You do not need to have a copy of MXNet's source code in order to run it, though you may download it
and map it your filesystem copy into the container if you wish (see examples below).

In addition to providing an reproducible environment (and producing a docker image that can be shared across
machines), an added benefit is being able to use docker volumes to dynamically set:

- the pretrained network
- the path to the output data that is captured
- the (VOC-like) dataset that is used for training

## Quick start

Build the container

```
$ docker build -t mxnet-rcnn mxnet-rcnn
```

Download VOC devkit and pretrained networks

```
$ mkdir -p pretrained-networks
$ cd pretrained-networks && wget http://data.dmlc.ml/models/imagenet/vgg/vgg16-0000.params && cd -
$ mkdir -p data
$ cd data && wget http://host.robots.ox.ac.uk/pascal/VOC/voc2012/VOCtrainval_11-May-2012.tar && tar -xf VOCtrainval_11-May-2012.tar && cd -
```

**Train** the network, capturing output to a directory.

```
$ mkdir -p training-runs/mxnet-rcnn-voc
$ nvidia-docker run --rm --name run-mxnet-rcnn-end2end \
  `#container volume mapping` \
  -v `pwd`/training-runs/mxnet-rcnn-voc:/media/ngv/output \
  -v `pwd`/pretrained-networks:/media/ngv/pretrained \
  -v `pwd`/data/VOCdevkit:/root/mxnet/example/rcnn/data/VOCdevkit \
  -it mxnet-rcnn \
  `# python script` \
  python train_end2end.py \
  --image_set 2012_train \
  --root_path /media/ngv/output \
  --pretrained /media/ngv/pretrained/vgg16 \
  --prefix /media/ngv/output/e2e \
  --gpus 0,1 \
  2>&1 | tee training-runs/mxnet-rcnn-voc/e2e-training-logs.txt
```

**Test** the network, using the parameters from the final epoch of training, capturing
the results to a directory of our choosing.


```
$ mkdir -p training-runs/mxnet-rcnn-voc/evaluate-on-voc/cache
$ nvidia-docker run --rm --name run-mxnet-rcnn-end2end \
  `#container volume mapping` \
  -v `pwd`/training-runs/mxnet-rcnn-voc:/media/ngv/output \
  -v `pwd`/data/VOCdevkit:/root/mxnet/example/rcnn/data/VOCdevkit \
  -v `pwd`/training-runs/mxnet-rcnn-voc/evaluate-on-voc:/root/mxnet/example/rcnn/data/VOCdevkit/results \
  -v `pwd`/training-runs/mxnet-rcnn-voc/evaluate-on-voc/cache:/media/ngv/output/cache \
  -it mxnet-rcnn \
  `# python script` \
  python test.py \
  --image_set 2012_val \
  --root_path /media/ngv/output \
  --prefix /media/ngv/output/e2e \
  --epoch 10 \
  --gpu 0 \
  2>&1 | tee /mnt/ngv/training-runs/2017-01-20-mxnet-rcnn-voc/e2e-testing-logs.txt
```

## Details

### What's built into the docker image

Instead of using [an off the shelf MXNet docker image](https://github.com/Kaixhin/dockerfiles/blob/master/cuda-mxnet/cuda_v8.0/Dockerfile)
we build off nvidia's [nvidia/cuda:8.0-cudnn5-devel](https://github.com/NVIDIA/nvidia-docker/blob/master/ubuntu-14.04/cuda/8.0/devel/cudnn5/Dockerfile)
and customize the build of MXNet to include additional operators required by the RCNN example.

We also specify a specific release to make the building of the image reproducible; if you wish a more recent build,
update or remove the git checkout command in the dockerfile, test it out, and then make a change to this dockerfile.

Finally, we install the required additional apt and pip dependencies, and build the RCNN dependencies by running
`make` within the directory.

(much of this could be accomplished by running the provided [additional_deps.sh](https://github.com/dmlc/mxnet/blob/master/example/rcnn/script/additional_deps.sh)
script.)

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
$ docker build -t mxnet-rcnn mxnet-rcnn
```

### Capturing output from training run against vanilla VOC

In this example, we create a directory to capture the output, and then map its path to `/media/ngv/output`
in order to capture the logs and network parameters from training. VOC is used for training by default.


```
$ mkdir -p training-runs/mxnet-rcnn-voc
$ nvidia-docker run --rm --name run-mxnet-rcnn-end2end \
  `#container volume mapping` \
  -v `pwd`/training-runs/mxnet-rcnn-voc:/media/ngv/output \
  -v /mnt/ngv/pretrained-networks:/media/ngv/pretrained \
  -v /mnt/ngv/datasets/VOC/VOCdevkit:/root/mxnet/example/rcnn/data/VOCdevkit \
  -it mxnet-rcnn \
  `# python script` \
  python train_end2end.py \
  --image_set 2012_trainval \
  --root_path /media/ngv/output \
  --pretrained /media/ngv/pretrained/vgg16 \
  --prefix /media/ngv/output/e2e \
  --gpus 0,1 \
  2>&1 | tee training-runs/mxnet-rcnn-voc/e2e-training-logs.txt
```


### Capturing output from training run against our own dataset in voc format

Same as previous example, but we also map in kitti in voc format from our local file system
to `/root/mxnet/example/rcnn/data/VOCdevkit`.

```
$ mkdir -p training-runs/mxnet-rcnn-voc
$ nvidia-docker run --rm --name run-mxnet-rcnn-end2end \
  `#container volume mapping` \
  -v `pwd`/training-runs/mxnet-rcnn-voc:/media/ngv/output \
  -v /mnt/ngv/pretrained-networks:/media/ngv/pretrained \
  -v /mnt/ngv/datasets/KITTI-VOC:/root/mxnet/example/rcnn/data/VOCdevkit \
  -it mxnet-rcnn \
  `# python script` \
  python train_end2end.py \
  --image_set 2012_trainval \
  --root_path /media/ngv/output \
  --pretrained /media/ngv/pretrained/vgg16 \
  --prefix /media/ngv/output/e2e \
  --gpus 0,1 \
  2>&1 | tee training-runs/mxnet-rcnn-voc/e2e-training-logs.txt
```

### Mapping in MXNet source code

In this example, we shadow the MXNet code with a copy from our filesystem. This can be handy if you wish
to alter / iterate on the code.

If necessary, clone MXNet (and make any modifications you'd like):

```
$ git clone git@github.com:dmlc/mxnet.git
```

Then map a subdirectory into the container to shadow `/root/mxnet/example/rcnn/rcnn/dataset`.

Note: since the build process stores compiled cython files within the source tree, it's not possible to map in
the entire mxnet source tree. When in doubt, shadow the individual files you wish to change.

```
$ nvidia-docker run --rm --name run-mxnet-rcnn-end2end \
  `#container volume mapping` \
  -v `pwd`/mxnet/example/rcnn/rcnn/dataset:/root/mxnet/example/rcnn/rcnn/dataset \
  -v `pwd`/mxnet:/root/mxnet \
  -v `pwd`/training-runs/mxnet-rcnn-voc:/media/ngv/output \
  -v /mnt/ngv/pretrained-networks:/media/ngv/pretrained \
  -v /mnt/ngv/datasets/VOC/VOCdevkit:/root/mxnet/example/rcnn/data/VOCdevkit \
  -it mxnet-rcnn \
  `# python script` \
  python train_end2end.py \
  --image_set 2012_trainval \
  --root_path /media/ngv/output \
  --pretrained /media/ngv/pretrained/vgg16 \
  --prefix /media/ngv/output/e2e \
  --gpus 0,1 \
  2>&1 | tee training-runs/mxnet-rcnn-voc/e2e-training-logs.txt
```
