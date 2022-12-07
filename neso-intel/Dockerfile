# Download base image ubuntu 22.04
FROM ubuntu:22.04
LABEL maintainer=""
LABEL version="0.0.1"
LABEL description="This is custom Docker Image that adds \
basic tools to Ubuntu 22.04"

# Disable Prompt During Packages Installation
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update
RUN apt-get install -y \
	git \
	vim \
	python3 \
	python3-pip \
	cmake \
	make \
	wget \
	rsync \
	curl \
    mpich \
    libmpich-dev \
    build-essential \
    gfortran \
    libncurses-dev \
    libncurses6 \
    libncursesw6 \
    libncurses5-dev \
    libncursesw5-dev \
    z3 \
    make \
    bzip2 \
    patch \
    unzip

## Install spack
RUN git clone -c feature.manyFiles=true https://github.com/spack/spack.git ~/spack
RUN cd ~/spack && git checkout releases/v0.18 

## find system packages
RUN . ~/spack/share/spack/setup-env.sh && spack external find
## find system compilers
RUN . ~/spack/share/spack/setup-env.sh && spack compiler find
## bootstrap spack
RUN pip3 install clingo
RUN . ~/spack/share/spack/setup-env.sh && spack --bootstrap find

## add system mpich and ncurses
RUN echo '  mpich:\n\
    externals:\n\
    - spec: mpich@4.0+hydra device=ch4 netmod=ofi\n\
      prefix: /usr\n\
      buildable: false\n\
  ncurses:\n\
    externals:\n\
    - spec: ncurses@6.3\n\
      prefix: /usr\n\
      buildable: false\n'\
        >> ~/.spack/packages.yaml

## clone NESO-Spack repo
RUN git clone https://github.com/ExCALIBUR-NEPTUNE/NESO-Spack ~/NESO-Spack
RUN cd ~/NESO-Spack && git checkout feature/neso-particles
## add NESO-Spack repo
RUN . ~/spack/share/spack/setup-env.sh && spack repo add ~/NESO-Spack
## build nektar++
RUN . ~/spack/share/spack/setup-env.sh && spack install intel-oneapi-compilers@2022.1.0
RUN . ~/spack/share/spack/setup-env.sh && spack load intel-oneapi-compilers && spack compiler find
RUN . ~/spack/share/spack/setup-env.sh && spack install neso ^dpcpp ^py-numpy%gcc

## clean spack downloads to try and reduce image size
RUN . ~/spack/share/spack/setup-env.sh && spack clean --downloads
