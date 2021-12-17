FROM nvidia/cuda:11.2.2-devel-ubuntu20.04 as base

# Dependencies for glvnd and X11.
RUN apt-get update \
  && apt-get install -y -qq --no-install-recommends \
    libglvnd0 \
    libgl1 \
    libglx0 \
    libegl1 \
    libxext6 \
    libx11-6 \
  && rm -rf /var/lib/apt/lists/*# Env vars for the nvidia-container-runtime.
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES graphics,utility,compute

# ADOP specific deps
RUN apt install -y -qq --no-install-recommends gcc-9 g++-9 xorg-dev git wget ca-certificates
RUN rm /usr/bin/g++ && rm /usr/bin/gcc
RUN ln -s /usr/bin/g++-9 /usr/bin/g++ && ln -s /usr/bin/gcc-9 /usr/bin/gcc
RUN update-ca-certificates
ENV PATH="/root/miniconda3/bin:${PATH}"
ARG PATH="/root/miniconda3/bin:${PATH}"
RUN wget --no-check-certificate https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh && bash ~/miniconda.sh -b -p /root/miniconda3 && echo "export PATH="/root/miniconda3/bin:$PATH"" >> ~/.bashrc && bash -c "source ~/.bashrc" 

FROM base as base.env
RUN git clone https://github.com/darglein/ADOP.git && cd ADOP && bash create_environment.sh 

FROM base.env as base.env.pytorch
RUN cd ADOP && bash install_pytorch.sh

FROM base.env.pytorch as final
RUN echo "source activate adop" >> ~/.bashrc
ENV PATH /root/miniconda3/bin:$PATH
RUN cd ADOP && git submodule update --init --recursive --jobs 0

RUN wget https://github.com/Kitware/CMake/releases/download/v3.22.1/cmake-3.22.1-linux-x86_64.sh \
      -q -O /tmp/cmake-install.sh \
      && chmod u+x /tmp/cmake-install.sh \
      && mkdir /usr/bin/cmake \
      && /tmp/cmake-install.sh --skip-license --prefix=/usr/bin/cmake \
      && rm /tmp/cmake-install.sh

ENV PATH="/usr/bin/cmake/bin:${PATH}"

ENV CONDA=${CONDA_PREFIX:-"$(dirname $(which conda))/../"}

RUN apt install -y make libjpeg-dev
RUN cd ADOP && \ 
export CC=gcc-9 && \
export CXX=g++-9 && \
export CUDAHOSTCXX=g++-9 && \
mkdir build

FROM final as deps
RUN echo "source activate adop" >> ~/.bashrc && \
cd /ADOP/build/ && \
cmake -DGLOG_PREFER_EXPORTED_GLOG_CMAKE_CONFIGURATION=FALSE -DBUILD_TESTING=OFF -DBUILD_EXAMPLES=OFF -DCMAKE_PREFIX_PATH="/root/miniconda3/envs/adop/lib/python3.9/site-packages/torch;/root/miniconda3/envs/adop" .. && \
make -j 8
