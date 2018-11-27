FROM alpine:latest
MAINTAINER https://github.com/brmather
ENV IMAGENAME=brmather/petsc4py
ENV PROJ_NAME=petsc4py-env

# Install things
RUN apk add --update --no-cache \
    build-base \
    linux-headers \
    python3-dev \
    py3-pip \
    py3-numpy \
    py3-scipy \
    py3-zmq \
    cmake \
    libzmq \
    zlib-dev \
    gfortran \
    g++ \
    musl-dev \
    lapack-dev \
    make \
    freetype-dev \
    libpng-dev \
    libxml2-dev \
    freeglut-dev


RUN python3 -m pip install --no-cache-dir --upgrade numpy
RUN python3 -m pip install --no-cache-dir setuptools wheel
RUN python3 -m pip install --no-cache-dir \
    matplotlib \
    packaging \
    jupyter \
    cartopy \
    lavavu


RUN alias python=python3

ENV PETSC_VERSION=petsc-3.10.2
RUN curl $PETSC_VERSION.tar.gz http://ftp.mcs.anl.gov/pub/petsc/release-snapshots/$PETSC_VERSION.tar.gz
RUN tar -xzf $PETSC_VERSION.tar.gz
WORKDIR /$PETSC_VERSION
RUN ./configure --with-shared-libraries --download-mpich \
                                        --download-hdf5 \
                                        --download-fblaslapack \
                                        --download-scalapack \
                                        --download-triangle \
                                        --download-mumps \
                                        --download-chaco \
                                        --download-ctetgen \
                                        --download-hypre \
                                        --download-eigen \
                                        --download-fftw \
                                        --download-parmetis \
                                        --download-metis \
                                        --prefix=/opt/petsc

RUN make PETSC_DIR=/$PETSC_VERSION PETSC_ARCH=arch-linux2-c-debug all
RUN make PETSC_DIR=/$PETSC_VERSION PETSC_ARCH=arch-linux2-c-debug install
RUN cd .. && rm -rf $PETSC_VERSION PETSC_VERSION.tar.gz

ENV PETSC_BIN=/opt/petsc/bin
RUN env MPICC=$PETSC_BIN/mpicc MPICXX=$PETSC_BIN/mpicxx MPIFC=$PETSC_BIN/mpifort python3 -m pip install --no-cache-dir mpi4py
RUN CC=$PETSC_BIN/mpicc FC=$PETSC_BIN/mpifort CXX=$PETSC_BIN/mpicxx HDF5_MPI="ON" HDF5_DIR=/opt/petsc python3 -m pip install --no-cache-dir --no-binary=h5py --no-deps h5py
RUN PETSC_DIR=/opt/petsc PETSC_ARCH=arch-linux-c-debug python3 -m pip install --no-cache-dir --no-deps petsc4py


# Install Tini.. this is required because CMD (below) doesn't play nice with notebooks for some reason:
# https://github.com/ipython/ipython/issues/7062, https://github.com/jupyter/notebook/issues/334
RUN curl -L https://github.com/krallin/tini/releases/download/v0.10.0/tini > tini && \
    echo "1361527f39190a7338a0b434bd8c88ff7233ce7b9a4876f3315c22fce7eca1b0 *tini" | sha256sum -c - && \
    mv tini /usr/local/bin/tini && \
    chmod +x /usr/local/bin/tini


# environment variable will internally run xvfb when glucifer is imported
# see /opt/underworld2/glucifer/__init__.py
ENV GLUCIFER_USE_XVFB 1

# add a notebook profile
RUN mkdir -p -m 700 /root/.jupyter/ && \
    echo "c.NotebookApp.ip = '*'" >> /root/.jupyter/jupyter_notebook_config.py

WORKDIR /

# expose notebook port
EXPOSE 8888


# note we also use xvfb which is required for viz
ENTRYPOINT ["/usr/local/bin/tini", "--"]

# a bit of tidying up
RUN apk del .build-dependencies