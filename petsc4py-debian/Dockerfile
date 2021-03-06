FROM debian:latest
MAINTAINER https://github.com/brmather
ENV IMAGENAME=brmather/petsc4py-debian
ENV PROJ_NAME=petsc4py-debian

RUN apt-get update -qq && \
    DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
        bash-completion \
        build-essential \
        gfortran \
        git \
        python3-dev \
        python3-pip \
        libxml2-dev \
        xorg-dev \
        ssh \
        libfreetype6-dev \
        libpng-dev \
        libxft-dev \
        xvfb \
        freeglut3-dev \
        libgl1-mesa-dri \
        libgl1-mesa-glx \
        rsync \
        vim \
        less \
        xauth \
        python3-tk \
        swig \
        gdb-minimal \
        cython \
        cmake \
        libgeos++ \
        libproj-dev \
        proj-bin \
        curl \
        wget && \
    apt-get clean && rm -rf /var/lib/apt/lists/*


RUN alias python=python3
# Upgrade and install python packages

RUN python3 -m pip install --no-cache-dir setuptools wheel && \
    python3 -m pip install --upgrade --no-cache-dir Cython numpy scipy matplotlib && \
    python3 -m pip install --no-cache-dir packaging \
    pyvirtualdisplay \
    jupyter \
    cartopy \
    lavavu 



# install PETSc
ENV PETSC_VERSION=petsc-3.10.2
RUN wget http://ftp.mcs.anl.gov/pub/petsc/release-snapshots/$PETSC_VERSION.tar.gz && \
    tar -xzf $PETSC_VERSION.tar.gz && cd $PETSC_VERSION && \
    ./configure COPTFLAGS='-O3 -march=native -mtune=native' \
                CXXOPTFLAGS='-O3 -march=native -mtune=native' \
                FOPTFLAGS='-O3 -march=native -mtune=native' \
                --with-shared-libraries \
                --with-debugging=0 \
                --download-mpich \
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
                --prefix=/opt/petsc && \
    make PETSC_DIR=/$PETSC_VERSION PETSC_ARCH=arch-linux2-c-opt all && \
    make PETSC_DIR=/$PETSC_VERSION PETSC_ARCH=arch-linux2-c-opt install && \
    cd ../ && rm -rf $PETSC_VERSION PETSC_VERSION.tar.gz

# # clean up
RUN apt-get remove -yq python && \
    apt autoremove -yq && \
    apt-get clean
    

# install mpi4py, h5py, petsc4py
ENV PETSC_BIN=/opt/petsc/bin
RUN env MPICC=$PETSC_BIN/mpicc MPICXX=$PETSC_BIN/mpicxx MPIFC=$PETSC_BIN/mpifort python3 -m pip install --no-cache-dir mpi4py && \
    CC=$PETSC_BIN/mpicc FC=$PETSC_BIN/mpifort CXX=$PETSC_BIN/mpicxx HDF5_MPI="ON" HDF5_DIR=/opt/petsc python3 -m pip install --no-cache-dir --no-binary=h5py --no-deps h5py && \
    PETSC_DIR=/opt/petsc PETSC_ARCH=arch-linux-c-debug python3 -m pip install --no-cache-dir --no-deps petsc4py


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