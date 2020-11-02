FROM golang:1.15.2 as builder

ARG SINGULARITY_COMMITISH="master"

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

FROM jupyter/datascience-notebook:4a112c0f11eb

USER root
COPY --from=builder /usr/local/singularity /usr/local/singularity
ENV PATH="/usr/local/singularity/bin:$PATH"
RUN apt-get update && apt-get install -y ca-certificates libseccomp2 \
   squashfs-tools fuse s3fs git-annex rclone httpfs2 \
    && rm -rf /tmp/*

RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
  -o "awscliv2.zip" && unzip awscliv2.zip \
  && ./aws/install && rm -rf ./aws awscliv2.zip

RUN wget https://github.com/kahing/goofys/releases/download/v0.24.0/goofys \
  && chmod +x goofys && mv goofys /usr/local/bin

USER $NB_USER

# RUN conda install --yes git-annex rclone && conda clean -tipsy
RUN pip install --no-cache-dir nbgitpuller nibabel dipy pybids pynidm pydra-ml \
  reproschema nda-tools datalad datalad-container datalad-neuroimaging
RUN pip install --no-cache-dir https://github.com/ReproNim/neurodocker/archive/master.zip
RUN pip install --no-cache-dir itkwidgets==0.32.0 jupytext nilearn plotly
RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager jupyter-matplotlib jupyterlab-datawidgets itkwidgets@0.32.0
RUN jupyter lab build
