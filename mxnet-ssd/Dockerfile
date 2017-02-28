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

# Clone MXNet repo and move into it
RUN cd /root && git clone --recursive https://github.com/dmlc/mxnet && cd mxnet && \
# use a known working point (includes key fixes to ssd on 2/17/2017 https://github.com/dmlc/mxnet/pull/4890)
  git checkout 2f222cfa143c5e444f5c6bbe550daa3a5af601ee && \
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
  sed -i 's/EXTRA_OPERATORS =/EXTRA_OPERATORS = example\/ssd\/operator/g' config.mk && \
# moar arch codes so this will run on dgx1
  sed -i 's/-gencode arch=compute_50,code=compute_50/-gencode arch=compute_50,code=compute_50 -gencode arch=compute_60,code=sm_60 -gencode arch=compute_61,code=sm_61 -gencode arch=compute_62,code=sm_62/g' config.mk && \
# Make
  make -j"$(nproc)"

# Install Python package
RUN cd /root/mxnet/python && python setup.py install

# Dependencies
RUN apt-get install -y python-opencv python-matplotlib python-numpy python-pip
RUN pip install easydict

WORKDIR /root/mxnet/example/ssd
