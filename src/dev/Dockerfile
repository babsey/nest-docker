FROM ubuntu:22.04
LABEL maintainer="s.graber@fz-juelich.de"

ARG NEST_VERSION=master
ARG SRC_PATH=/tmp
ARG CMAKE_C_COMPILER_LAUNCHER=ccache
ARG CMAKE_CXX_COMPILER_LAUNCHER=ccache
ARG CXX_FLAGS="-pedantic -Wextra -Wno-unknown-pragmas -D_GLIBCXX_ASSERTIONS"

ENV TERM=xterm \
    TZ=Europe/Berlin \
    DEBIAN_FRONTEND=noninteractive \
    PATH=/root/.local/bin:${PATH}

RUN apt-get update && apt-get install -y --no-install-recommends \
    automake \
    autotools-dev \
    build-essential \
    ccache \
    cmake \
    curl \
    cython3 \
    freeglut3-dev \
    git \
    gosu \
    jq \
    less \
    libboost-filesystem-dev libboost-regex-dev libboost-wave-dev \
    libboost-python-dev libboost-program-options-dev libboost-test-dev \
    libboost-dev \
    libgomp1 \
    libgsl-dev  \
    libgsl0-dev \
    libhdf5-dev \
    libltdl-dev \
    libltdl7 \
    libmusic1v5 \
    libncurses-dev \
    libomp-dev \
    libopenmpi-dev \
    libpcre3 \
    libpcre3-dev \
    libpython3.10 \
    libreadline-dev \
    libreadline8 \
    libtool \
    libzmq3-dev \
    llvm-dev \
    nano \
    openmpi-bin \
    openssh-client \
    openssh-server \
    pandoc \
    pep8 \
    pkg-config \
    python3-dev \
    python3-flask \
    python3-flask-cors \
    python3-h5py \
    python3-ipython \
    python3-junitparser \
    python3-jupyter-core \
    python3-matplotlib \
    python3-mpi4py \
    python3-nose \
    python3-pandas \
    python3-path \
    python3-pip \
    python3-pytest \
    python3-pytest-timeout \
    python3-pytest-xdist \
    python3-restrictedpython \
    python3-setuptools \
    python3-sphinx \
    python3-statsmodels \
    python3-sympy \
    python3-tk \
    python3-tqdm \
    tcl8.6 \
    tcl8.6-dev \
    tk8.6-dev \
    uwsgi \
    vera++ \
    wget && \
    apt-get autoremove

RUN python3 -m pip install --upgrade scipy numpy numpydoc
RUN python3 -m pip install --upgrade pip setuptools wheel mock
RUN python3 -m pip install --force-reinstall --upgrade --no-binary=h5py h5py
RUN ldconfig

# Download NEST
RUN cd ${SRC_PATH} && \
    wget https://github.com/nest/nest-simulator/archive/refs/heads/${NEST_VERSION}.tar.gz -P ${SRC_PATH} && \
    tar -xzf ${NEST_VERSION}.tar.gz

# Installing the requirements_docs.txt stops at some time with an error
# Installing every package fix it.
# RUN python3 -m pip install sphinx>=6.2.1 sphinx_rtd_theme sphinx_autobuild sphinx_gallery sphinx-tabs
# RUN python3 -m pip install sphinx_design sphinx-material sphinx-copybutton sphinx-carousel
# RUN python3 -m pip install sphinx-notfound-page sphinxcontrib-mermaid sphinxcontrib-plantuml nbsphinx
# RUN python3 -m pip install example Image breathe csvkit docutils PyYAML>=4.2b1 tqdm yamllint

RUN python3 -m pip install -r ${SRC_PATH}/nest-simulator-${NEST_VERSION}/requirements.txt

# RUN python3 -m pip install sphinx_gallery==0.10.1

    # Install music
RUN chmod +x ${SRC_PATH}/nest-simulator-${NEST_VERSION}/build_support/install_music.sh && \
    ${SRC_PATH}/nest-simulator-${NEST_VERSION}/build_support/install_music.sh

# # Install libneurosim
# RUN cd ${SRC_PATH}/nest-simulator-${NEST_VERSION} && \
#     PYLIB_DIR="$(python3 -c "import sysconfig; print(sysconfig.get_path('include'))" | sed 's/include/lib/')" && \
#     chmod +x ${SRC_PATH}/nest-simulator-${NEST_VERSION}/build_support/install_csa-libneurosim.sh && \
#     ${SRC_PATH}/nest-simulator-${NEST_VERSION}/build_support/install_csa-libneurosim.sh $PYLIB_DIR
# # Install sionlib
# RUN chmod +x ${SRC_PATH}/nest-simulator-${NEST_VERSION}/build_support/install_sionlib.sh && \
#     ${SRC_PATH}/nest-simulator-${NEST_VERSION}/build_support/install_sionlib.sh
# Install python requirements
# RUN pip install -r ${SRC_PATH}/nest-simulator-${NEST_VERSION}/requirements.txt

# Build nest
RUN mkdir ${SRC_PATH}/nest-build && cd $_ && \
    cmake -DCMAKE_INSTALL_PREFIX:PATH=/opt/nest \
          -Dwith-optimize="-O2" \
          -Dwith-warning=ON \
          -Dwith-userdoc=ON \
          -Dwith-boost=ON \
          -Dwith-ltdl=ON \
          -Dwith-gsl=ON \
          -Dwith-readline=ON \
          -Dwith-python=ON \
          -Dwith-mpi=ON \
          -Dwith-openmp=ON \
          -Dwith-libneurosim=OFF \
          -Dwith-sionlib=OFF \
          -Dwith-music=ON \
          -Dwith-hdf5=ON \
          ${SRC_PATH}/nest-simulator-${NEST_VERSION} && \
          make && \
          make install

# Install NESTML and more
RUN python3 -m pip install --upgrade pip && \
    python3 -m pip install nest-desktop --pre && \
    python3 -m pip uninstall nestml -y && \
    python3 -m pip install --upgrade https://github.com/nest/nestml/archive/refs/heads/master.zip && \
    python3 -m pip install nestml-server@git+https://github.com/babsey/nestml-server@v1.0-b4

COPY entrypoint.sh /usr/local/bin/entrypoint.sh
RUN chmod +x /usr/local/bin/entrypoint.sh

COPY test-nest.sh /opt/test-nest.sh
RUN chmod +x /opt/test-nest.sh

EXPOSE 8080 52425 52426 54286
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
