FROM nielsbohr/python-cuda-notebook:latest
LABEL MAINTAINER="Rasmus Munk <rasmus.munk@nbi.ku.dk>"
ARG PACKAGE_TIMEOUT=60

USER root

# Setup default timeout of installing packages
RUN echo "[global]" > /etc/pip.conf \
    && echo "timeout=$PACKAGE_TIMEOUT" >> /etc/pip.conf

RUN apt-get update \
    && apt-get install -yq --no-install-recommends \
    graphviz \
    libgl1-mesa-glx \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /tmp

USER $NB_UID

RUN conda config --set remote_read_timeout_secs $PACKAGE_TIMEOUT

# Packages
COPY requirements.txt /tmp/requirements.txt
RUN conda run -n python3 pip install -r requirements.txt \
    && conda clean -y -f --all \
    && fix-permissions $CONDA_DIR \
    && fix-permissions /home/$NB_USER

RUN conda run -n python3 pip install hdbscan --no-build-isolation --no-binary :all:

WORKDIR /home/$NB_USER
# Run container as
USER $NB_UID

