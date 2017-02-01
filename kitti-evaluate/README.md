# Evaluate KITTI

A container to help evaluate object detections ala
[KITTI](http://www.cvlibs.net/datasets/kitti/eval_object.php).

The original `evaluate_object.cpp` provided by the development kit (mirrored here for reference) is copied into
two versions, one for evaluating 50% overlap and the other for evaluating 70% overlap. In addition, tweaks are made
to utput mean average precision for easy, medium and hard bounding boxes and to avoid deleting
the directory containing the labels to evaluate. The `Dockerfile` builds these files into respective executables.

## Example usage:

Build the container:

```
$ docker build -t kitti-evaluate kitti-evaluate
```

Assuming that:

- The ground truth labels reside at `/media/datasets/KITTI/training/label_2`
- The labels produced when evaluating our own network are at `my-kitti-labels`

and that directories contain KITTI-style label files for each image in the test set:

```
$ ls my-kitti-labels | head -3
000000.txt
000001.txt
000002.txt
```

```
$ ls /media/datasets/KITTI/training/label_2 | head -3
000000.txt
000001.txt
000002.txt
```

Here's how to evaluate your labels for 50% overlap:

```
$ mkdir -p kitti-eval/my-kitti-labels/evaluate50
$ docker run --rm \
  -v /media/datasets/KITTI/training/label_2:/root/data/object/label_2:ro \
  -v `pwd`/my-kitti-labels:/root/results/my-kitti-labels/data:ro \
  -v `pwd`/kitti-eval/my-kitti-labels/evaluate50:/root/results/my-kitti-labels \
  -it kitti-evaluate ./evaluate_object50 my-kitti-labels
Thank you for participating in our evaluation!
Loading detections...
  done.
Mean AP Easy 0.811846
Mean AP Moderate 0.645078
Mean AP Hard 0.550397
PDFCROP 1.38, 2012/11/02 - Copyright (c) 2002-2012 by Heiko Oberdiek.
==> 1 page written on `car_detection.pdf'.
Your evaluation results are available at:
http://www.cvlibs.net/datasets/kitti/user_submit_check_login.php?benchmark=object&user=(null)&result=my-kitti-labels
```

(note that the evaluation results are not actually submitted and no email is actually sent).

Results are stored:

```
$ find kitti-eval/my-kitti-labels/evaluate50
kitti-eval/my-kitti-labels/evaluate50
kitti-eval/my-kitti-labels/evaluate50/plot
kitti-eval/my-kitti-labels/evaluate50/plot/car_detection.txt
kitti-eval/my-kitti-labels/evaluate50/plot/car_detection.pdf
kitti-eval/my-kitti-labels/evaluate50/plot/car_detection.eps
kitti-eval/my-kitti-labels/evaluate50/plot/car_detection.png
kitti-eval/my-kitti-labels/evaluate50/plot/car_detection.gp
kitti-eval/my-kitti-labels/evaluate50/stats_car_detection.txt
kitti-eval/my-kitti-labels/evaluate50/data
kitti-eval/my-kitti-labels/evaluate50/stats_car_mAP.txt
```

Evaluation with 70% overlap can be similarly run as follows:

```
$ mkdir -p kitti-eval/my-kitti-labels/evaluate70
$ docker run --rm \
  -v /media/datasets/KITTI/training/label_2:/root/data/object/label_2:ro \
  -v `pwd`/my-kitti-labels:/root/results/my-kitti-labels/data:ro \
  -v `pwd`/kitti-eval/my-kitti-labels/evaluate70:/root/results/my-kitti-labels \
  -it kitti-evaluate ./evaluate_object70 my-kitti-labels
```