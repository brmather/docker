##
# Image for pygplates Jupyter server
#

FROM ubuntu:18.04

# install dependencies for pygplates
RUN apt-get update -qq && \
    DEBIAN_FRONTEND=noninteractive apt-get install -yq --no-install-recommends \
    libglew-dev \
    python2.7-dev \
    python-pip \
    libboost-dev \
    libboost-python-dev \
    libboost-python1.65.1 \
    libboost-thread-dev \
    libboost-program-options-dev \
    libboost-test-dev \
    libboost-system-dev \
    libqt4-dev \
	libgdal-dev \
    libgdal20 \
    gdal-abi-2-2-3 \
	libcgal-dev \
    libgeos-dev \
	libproj-dev \
	libqwt-dev \
	libxrender-dev \
    libice-dev \
    libsm-dev \
    libfreetype6-dev \
    libfontconfig1-dev \
	git \
	wget \
    g++ \
	gcc && \
	apt-get remove -yq python3 && \
    apt-get autoremove -yq && \
	apt-get clean && rm -rf /var/lib/apt/lists/*




# install all the python and ipython notebook requirements
RUN python2 -m pip install --no-cache-dir setuptools wheel Cython && \
    python2 -m pip install --no-cache-dir --upgrade \
    numpy \
    scipy \
    matplotlib \
    pandas \
    sympy \
    boost \
    pillow \
    cartopy \
    nose && \
    python2 -m pip install git+https://github.com/matplotlib/basemap.git && \
    python2 -m pip install --no-cache-dir jupyter ipyparallel && \
    rm -rf /tmp/pip-*


# use wget to get the correct pygplates package from sourceforge
RUN wget https://sourceforge.net/projects/gplates/files/pygplates/beta-revision-18/pygplates-ubuntu-bionic_2.1_1_amd64.deb && \
    dpkg -i pygplates-ubuntu-bionic_2.1_1_amd64.deb


# remove python3 installation
# RUN apt-get remove -yq python3 && \
#     apt-get autoremove -yq && \
#     apt-get clean

# Install non-linear optimisation package, with options to import into python
#RUN wget http://ab-initio.mit.edu/nlopt/nlopt-2.4.2.tar.gz
#RUN tar -xvzf nlopt-2.4.2.tar.gz
#RUN cd nlopt-2.4.2 && ./configure --enable-shared && make && make install

# Set python path to find pygplates and nlopt

RUN git clone https://github.com/tonysyu/mpltools.git /mpltools
RUN cd /mpltools && python setup.py install

RUN export LD_LIBRARY_PATH=/usr/local/lib

RUN useradd -ms /bin/bash ipython 

ENV NB_USER ipython

USER root

WORKDIR /home/$NB_USER/work
COPY jupyter_notebook_config.py /home/$NB_USER/.jupyter/
RUN chown -R $NB_USER:root /home/$NB_USER/.jupyter

# Add Tini
ENV TINI_VERSION v0.9.0
ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /tini
RUN chmod +x /tini
ENTRYPOINT ["/tini", "--"]

CMD ["/home/ipython/work/ipython_notebook.sh"]

Env PYTHONPATH ${PYTHONPATH}:/usr/lib:/usr/lib/pygplates/revision18/
ADD . /home/ipython/work/

RUN chmod +x /home/$NB_USER/work/ipython_notebook.sh
RUN chown -R $NB_USER:root /home/$NB_USER/work

USER ipython
