# MXNet RFCN

Builds a docker image for [MXNet RFCN](https://github.com/terrychenism/mx-rfcn).

```
$ docker build -t mxnet-rfcn mxnet-rfcn
```

Example training command using docker volumes (assuming you have the appropriate pre-trained network downloaded
and the VOC devkit downloaded):

```
$ nvidia-docker run --rm \
  -v `pwd`/training-runs/mxnet-rfcn-voc:/media/ngv/output \
  -v `pwd`/pretrained-networks:/media/ngv/pretrained:ro \
  -v `pwd`/data/VOCdevkit:/root/mxnet/example/rfcn/data/VOCdevkit:ro \
  -v /home/krosaen/dev/container-fns/fns/mxnet-rfcn-train/static/root/mxnet/example/rfcn/train_end2end_resnext.py:/root/mxnet/example/rfcn/train_end2end_resnext.py:ro \
  -v /home/krosaen/dev/container-fns/fns/mxnet-rfcn-train/static/root/mxnet/example/rfcn/utils/load_data.py:/root/mxnet/example/rfcn/utils/load_data.py:ro \
  -v /home/krosaen/dev/container-fns/fns/mxnet-rfcn-train/static/root/mxnet/example/rfcn/helper/dataset/pascal_voc.py:/root/mxnet/example/rfcn/helper/dataset/pascal_voc.py:ro \
  -it mxnet-rfcn \
  python train_end2end_resnext.py \
  --image_set trainval \
  --root_path /media/ngv/output \
  --outdata-path /media/ngv/output \
  --year 2012 \
  --test_image_set '' \
  --pretrained /media/ngv/pretrained/resnext-101 \
  --prefix /media/ngv/output/faster-resnext-101 \
  --lr 0.001 \
  --num_epoch 15 \
  --gpus 0 \
  2>&1 | tee -a training-runs/mxnet-rfcn-voc/training-logs.txt

```

See the [MXNet RCNN dockerfile](https://github.com/umautobots/nn-dockerfiles/tree/master/mxnet-rcnn) for a better documented example.