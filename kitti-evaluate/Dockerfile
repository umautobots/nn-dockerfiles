FROM ubuntu:14.04

RUN apt-get update && apt-get install -y \
  build-essential \
  gnuplot \
  ghostscript \
  texlive-extra-utils

COPY cpp /root/cpp
WORKDIR /root
RUN cd /root && \
  g++ cpp/evaluate_object50.cpp -O3 -o evaluate_object50 && \
  g++ cpp/evaluate_object70.cpp -O3 -o evaluate_object70


