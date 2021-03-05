FROM golang:1.15.2 as builder

ARG SINGULARITY_COMMITISH="v3.7.1"

WORKDIR $GOPATH/src/github.com/sylabs
RUN apt-get update && apt-get install -y \
    build-essential \
    uuid-dev \
    libgpgme-dev \
    squashfs-tools \
    libseccomp-dev \
    wget \
    pkg-config \
    git \
    cryptsetup-bin
RUN git clone https://github.com/sylabs/singularity.git \
    && cd singularity \
    && git checkout "$SINGULARITY_COMMITISH" \
    && ./mconfig -p /usr/local/singularity \
    && cd builddir \
    && make \
    && make install

FROM jupyter/datascience-notebook:016833b15ceb

USER root
COPY --from=builder /usr/local/singularity /usr/local/singularity
ENV PATH="/usr/local/singularity/bin:$PATH"
RUN apt-get update && apt-get install -y ca-certificates libseccomp2 \
   squashfs-tools fuse s3fs netbase less parallel \
    && rm -rf /tmp/*

RUN curl --silent --show-error "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
  -o "awscliv2.zip" && unzip awscliv2.zip \
  && ./aws/install && rm -rf ./aws awscliv2.zip

RUN wget --quiet https://github.com/kahing/goofys/releases/download/v0.24.0/goofys \
  && chmod +x goofys && mv goofys /usr/local/bin

USER $NB_USER

RUN pip install --no-cache-dir itkwidgets==0.32.0 jupytext plotly pyvista
RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager \
  jupyter-matplotlib jupyterlab-datawidgets itkwidgets@0.32.0 jupyterlab-plotly \
  plotlywidget jupyter-threejs --no-build \
  && export NODE_OPTIONS=--max-old-space-size=4096 \
  && jupyter lab build && rm -rf /tmp/*

RUN conda install --yes datalad rclone && conda clean -tipsy
RUN wget --quiet https://raw.githubusercontent.com/DanielDent/git-annex-remote-rclone/v0.6/git-annex-remote-rclone \
  && chmod +x git-annex-remote-rclone && mv git-annex-remote-rclone /opt/conda/bin
RUN pip install --no-cache-dir nbgitpuller nibabel dipy pybids pynidm pydra-ml \
  reproschema nda-tools datalad datalad-container datalad-neuroimaging pyontutils \
  datalad-osf PyMARE git+https://github.com/ReproNim/reproman.git@v0.4.1 gower \
  nilearn zarr
RUN pip install --no-cache-dir \
  git+https://github.com/nipype/pydra.git \
  git+https://github.com/ReproNim/neurodocker.git@f104933

USER root
RUN apt-get update && apt-get install -y tmux \
    && rm -rf /tmp/*
USER $NB_USER