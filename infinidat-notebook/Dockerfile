# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.

# A Dockerfile for use by Infinidat Professional Services.
FROM jupyter/base-notebook

LABEL maintainer="David Ohlemacher <dohlemacher@infinidat.com>"

USER root

# Install all OS dependencies for fully functional notebook server
RUN apt-get update && apt-get install -yq --no-install-recommends \
    build-essential \
    curl \
    git \
    inkscape \
    jed \
    libsm6 \
    libxext-dev \
    libxrender1 \
    lmodern \
    nano \
    netcat \
    pandoc \
    python-dev \
    rsync \
    ssh \
    texlive-fonts-extra \
    texlive-fonts-recommended \
    texlive-generic-recommended \
    texlive-latex-base \
    texlive-latex-extra \
    texlive-xetex \
    tmux \
    tzdata \
    unzip \
    vim \
    && apt purge \
    && apt clean \
    && rm -rf /var/lib/apt/lists/*

# Kubectl CLI
RUN \
    apt update \
    && apt install -yq gnupg2 apt-utils apt-transport-https \
    && curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - \
    && echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list \
    && apt update \
    && apt install -yq kubectl \
    && apt purge \
    && apt clean \
    && rm -rf /var/lib/apt/lists/*

# Helm CLI. Using a list and bash since pushd is not available in sh.
RUN ["/bin/bash", "-c", "mkdir /tmp/helm \
    && pushd /tmp/helm > /dev/null \
    && wget --quiet https://get.helm.sh/helm-v2.13.1-linux-amd64.tar.gz \
    && tar xfz helm-v2.13.1-linux-amd64.tar.gz \
    && chmod +x linux-amd64/helm \
    && mv linux-amd64/helm /usr/local/bin \
    && popd > /dev/null \
    && rm -rf /tmp/helm"]

# Fix sudoers and set a passwd
ARG NB_USER_PASSWD=jupyter
COPY 020_sudo_for_jovyan /etc/sudoers.d/
RUN echo "$NB_USER:$NB_USER_PASSWD" | chpasswd

# Install bash_kernel.
RUN conda install --quiet --yes -c conda-forge bash && \
     conda clean --all -f -y && \
	 pip install bash_kernel && \
	 python -m bash_kernel.install && \
     rm -rf /home/$NB_USER/.local && \
     fix-permissions $CONDA_DIR && \
     fix-permissions /home/$NB_USER

# Tweak Notebooks
# Ref:
#    - https://towardsdatascience.com/bringing-the-best-out-of-jupyter-notebooks-for-data-science-f0871519ca29
RUN \
    pip install --no-cache-dir jupyterthemes
RUN pip install --no-cache-dir jupyter_contrib_nbextensions
# --user
ENV PATH="/home/$NB_USER/.local/bin:${PATH}"
RUN pip install --no-cache-dir ipywidgets
RUN pip install --no-cache-dir nbgitpuller
RUN jupyter contrib nbextension install
RUN jupyter nbextension enable --py widgetsnbextension

RUN jt -t gruvboxd \
    && conda install -c conda-forge rise

# Always switch back to NB_UID.
USER $NB_UID
