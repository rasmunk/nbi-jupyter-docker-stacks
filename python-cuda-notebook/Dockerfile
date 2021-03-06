FROM nielsbohr/python-notebook:latest
LABEL MAINTAINER="Rasmus Munk <rasmus.munk@nbi.ku.dk>"
ARG PACKAGE_TIMEOUT=60

USER root

WORKDIR /tmp

ARG CUDA_OS_DIST=ubuntu2004
ARG CUDA_CPU=x86_64
ARG CUDA_MAJOR_VERSION=11
ARG CUDA_MIN_VERSION=0
ENV CUDA_VERSION=${CUDA_MAJOR_VERSION}.${CUDA_MIN_VERSION}
ARG CUDA_PKG_VERSION=11-0

ARG NCCL_VERSION=2.9.6
ARG CUDNN_VERSION=8.0.5.39

ENV CUDA_PATH="/usr/local/cuda"
ENV CUDA_11_0_PATH="/usr/local/cuda-11.0"

RUN apt-get update && apt-get install -y --no-install-recommends \
    gnupg2 \
    ca-certificates \
    && curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/${CUDA_OS_DIST}/${CUDA_CPU}/7fa2af80.pub | apt-key add - \
    && echo "deb https://developer.download.nvidia.com/compute/cuda/repos/${CUDA_OS_DIST}/${CUDA_CPU} /" > /etc/apt/sources.list.d/cuda.list \
    && echo "deb https://developer.download.nvidia.com/compute/machine-learning/repos/${CUDA_OS_DIST}/${CUDA_CPU} /" > /etc/apt/sources.list.d/nvidia-ml.list \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# For libraries in the cuda-compat-* package: https://docs.nvidia.com/cuda/eula/index.html#attachment-a
RUN apt-get update && apt-get install -y --no-install-recommends \
    cuda-cudart-11-0=11.0.221-1 \
    cuda-compat-11-0 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Runtime and NCCL
RUN apt-get update && apt-get install -y --no-install-recommends \
    cuda-libraries-11-0=11.0.3-1 \
    libnpp-11-0=11.1.0.245-1 \
    cuda-nvtx-11-0=11.0.167-1 \
    libcublas-11-0=11.2.0.252-1 \
    libcusparse-11-0=11.1.1.245-1 \
    libnccl2=$NCCL_VERSION-1+cuda11.0 \
    && apt-get clean \
    && rm -fr /var/lib/apt/lists/* \
    && apt-mark hold libcublas-11-0 libnccl2

# CUDNN
RUN apt-get update && apt-get install -y --no-install-recommends \
    libcudnn8=$CUDNN_VERSION-1+cuda11.0 \
    && apt-mark hold libcudnn8 && \
    rm -rf /var/lib/apt/lists/*

# Add CUDA symlink and missing symlink for libcusolver
RUN ln -s /usr/local/cuda-${CUDA_VERSION} /usr/local/cuda \
    && ln -s /usr/local/cuda/lib64/libcusolver.so.10 /usr/local/cuda/lib64/libcusolver.so.11 \
    && ln -s /usr/local/cuda/lib64/libcusolverMg.so.10 /usr/local/cuda/lib64/libcusolverMg.so.11

ENV PATH="${CUDA_PATH}/bin:${PATH}"
ENV LD_LIBRARY_PATH="${CUDA_PATH}/lib64"

# nvidia-container-runtime
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES compute,utility
ENV NVIDIA_REQUIRE_CUDA "cuda>=11.0 brand=tesla,driver>=418,driver<419 brand=tesla,driver>=440,driver<441 driver>=450"

# set tensorflow verbosity to error level, (default is 0/everything)
ENV TF_CPP_MIN_LOG_LEVEL="3"

RUN chown $NB_UID:$NB_GID -R /tmp

# Install user packages
USER $NB_UID

COPY requirements.txt /tmp/
RUN conda run -n python3 pip install -r requirements.txt \
    && conda clean --all -f -y \
    && fix-permissions $CONDA_DIR \
    && fix-permissions /home/$NB_USER

WORKDIR /home/$NB_USER

# Ensure that container Runs as
USER $NB_UID

