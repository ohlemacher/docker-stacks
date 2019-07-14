# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
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
    && rm -rf /var/lib/apt/lists/*

RUN \
    apt update \
    && apt install -yq gnupg2 apt-utils apt-transport-https \
    && curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - \
    && echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list \
    && apt update \
    && apt install -yq kubectl \
    && rm -rf /var/lib/apt/lists/*


ARG NB_USER_PASSWD=jupyter
COPY 020_sudo_for_jovyan /etc/sudoers.d/
RUN echo "$NB_USER:$NB_USER_PASSWD" | chpasswd


# #RUN usermod -u $NB_ID  $NB_USER;
# # RUN ["/bin/bash", "-c", "set -o pipefail && wget -O - https://some.site | wc -l > /number"]
# RUN \
#     set -o pipefail \
#     && usermod -g $NB_GID $NB_USER \
#     && echo -e $(id -a) > /home/jovyan/id.txt \
#     && usermod -aG sudo   $NB_USER \
#     && echo "$NB_USER:$NB_USER_PASSWD" | chpasswd \
#     && echo -e "$NB_USER\tALL=\(ALL\)\tNOPASSWD: ALL" > "/etc/sudoers.d/020_sudo_for_$NB_USER"
# Fix sudoers
#RUN sed 's?#%sudo  ALL=(ALL:ALL) ALL?%sudo  ALL=(ALL:ALL) ALL?' </etc/sudoers >/tmp/sudoers \
#    && mv /tmp/sudoers /etc/sudoers \
#    && chmod 440 /etc/sudoers
#RUN echo '%sudo  ALL=(ALL:ALL) ALL' | sudo EDITOR='tee -a' visudo

# Tweak Notebooks
# Ref:
#    - https://towardsdatascience.com/bringing-the-best-out-of-jupyter-notebooks-for-data-science-f0871519ca29
RUN \
    pip install jupyterthemes
RUN jt -t gruvboxd
RUN pip install jupyter_contrib_nbextensions --user
ENV PATH="/home/$NB_USER/.local/bin:${PATH}"
RUN jupyter contrib nbextension install
RUN pip install ipywidgets
RUN jupyter nbextension enable --py widgetsnbextension
RUN chown -R $NB_UID:$NB_GID /home/$NB_USER

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID

