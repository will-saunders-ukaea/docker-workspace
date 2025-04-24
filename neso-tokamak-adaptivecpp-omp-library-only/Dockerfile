# syntax=docker/dockerfile:1
#Download base image ubuntu 22.04
FROM ubuntu:24.04
LABEL maintainer="will.saunders@@ukaea.uk"
LABEL version="0.0.1"
LABEL description="This is custom Docker Image that adds \
basic tools to Ubuntu 24.04"

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
    build-essential \
    gfortran \
    z3 \
    make \
    bzip2 \
    patch \
    unzip

## Install spack
RUN git clone -c feature.manyFiles=true https://github.com/spack/spack.git ~/spack
RUN cd ~/spack && git checkout releases/v0.23

## find system compilers
RUN . ~/spack/share/spack/setup-env.sh && spack compiler find

ARG NUM_BUILD_WORKERS=32

## clone NESO-Tokamak
RUN git clone --recurse-submodules https://github.com/ExCALIBUR-NEPTUNE/NESO-Tokamak.git ~/NESO-Tokamak

RUN mkdir ~/NESO-Tokamak/spack_gcc
COPY <<"EOT" /root/NESO-Tokamak/spack_gcc/spack.yaml
spack:
  specs:
  - neso-tokamak%gcc ^adaptivecpp ^mpich ^neso.adaptivecpp
  view:
    gcc-adaptivecpp:
      root: views/gcc-adaptivecpp
      select: ['%gcc']
      link_type: symlink
  concretizer:
    unify: when_possible
  repos:
  - ../spack/repo
  - ../NESO/NESO-Spack
  - $spack/var/spack/repos/builtin
  develop:
    nektar:
      path: ../NESO/nektar
      spec: nektar@master
    neso:
      path: ../NESO
      spec: neso@working
    neso-particles:
      path: ../NESO/neso-particles
      spec: neso-particles@working
    neso-tokamak:
      path: ../
      spec: neso-tokamak@working
EOT

RUN . ~/spack/share/spack/setup-env.sh && spack env activate -d ~/NESO-Tokamak/spack_gcc && spack concretize
RUN . ~/spack/share/spack/setup-env.sh && spack env activate -d ~/NESO-Tokamak/spack_gcc && spack install -j $NUM_BUILD_WORKERS




