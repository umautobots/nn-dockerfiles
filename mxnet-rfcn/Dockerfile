FROM nvidia/cuda:8.0-cudnn5-devel

# mxnet
#
# note: we build it ourselves instead of using FROM kaixhin/cuda-mxnet:8.0 to control
# the version of mxnet we are building. Bleeding edge master is not always fun.
#

# Install git, wget and other dependencies
RUN apt-get update && apt-get install -y \
  git \
  libopenblas-dev \
  libopencv-dev \
  python-dev \
  python-numpy \
  python-setuptools \
  wget

# Clone MXNet fork and move into it
# https://github.com/terrychenism/mx-rfcn
RUN cd /root && git clone --recursive https://github.com/terrychenism/mxnet.git && cd mxnet && \
# use a known working commit that includes latest fixes we contributed to rcnn example code https://github.com/dmlc/mxnet/pull/4840
  git checkout rfcn-rebase  && \
# Copy config.mk
  cp make/config.mk config.mk && \
# Set OpenBLAS
  sed -i 's/USE_BLAS = atlas/USE_BLAS = openblas/g' config.mk && \
# Set CUDA flag
  sed -i 's/USE_CUDA = 0/USE_CUDA = 1/g' config.mk && \
  sed -i 's/USE_CUDA_PATH = NONE/USE_CUDA_PATH = \/usr\/local\/cuda/g' config.mk && \
# Set cuDNN flag
  sed -i 's/USE_CUDNN = 0/USE_CUDNN = 1/g' config.mk && \
# extra operators needed by rcnn exaples
  sed -i 's/EXTRA_OPERATORS =/EXTRA_OPERATORS = example\/rcnn\/operator/g' config.mk && \
# moar arch codes so this will run on dgx1
  sed -i 's/-gencode arch=compute_50,code=compute_50/-gencode arch=compute_50,code=compute_50 -gencode arch=compute_60,code=sm_60 -gencode arch=compute_61,code=sm_61 -gencode arch=compute_62,code=sm_62/g' config.mk && \
# Make
  make -j"$(nproc)"


# Install Python package
RUN cd /root/mxnet/python && python setup.py install

# clone r-fcn scripts into sample directory
RUN cd /root/mxnet/example && \
    git clone https://github.com/terrychenism/mx-rfcn rfcn

# opencv
#
# Note: we used to build and install opencv3 but that started causing conflicting library issues because
# opencv 2.7 is included via libopencv-dev above. Simply installying the python bindings to the same library
# seems to be sufficient.

RUN apt-get install -y python-opencv

# random other dependencies

RUN apt-get install -y python-scipy \
  python-matplotlib \
  python-pip

RUN pip install easydict scikit-image


WORKDIR /root/mxnet/example/rfcn

