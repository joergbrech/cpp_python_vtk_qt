FROM ubuntu:20.04
MAINTAINER Jan Kleinert <jan.kleinert@dlr.de>
LABEL Description="Ubuntu 20.04 with Qt5, VTK setup for use with pybind11, python and some dev tools"
SHELL ["/bin/bash", "-c"]

ARG qt_version=5.14.1
ARG python_version=3.7
ARG vtk_version=9.0.1

ENV DEBIAN_FRONTEND noninteractive
ENV PATH=/usr/local/Qt-${qt_version}/bin:$PATH
ENV LIBRARY_PATH=/usr/local/lib:$LIBRARY_PATH
ENV LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
ENV PYTHONPATH "${PYTHONPATH}:/usr/local/lib/python${python_version}/site-packages"

COPY requirements.txt requirements.txt

#######################################################
# Install software requirements, build tools, python  #
#######################################################

RUN apt-get update -y && apt-get upgrade -y; \
# Prepare apt-get build-dep (https://askubuntu.com/questions/496549/error-you-must-put-some-source-uris-in-your-sources-list)
    cp /etc/apt/sources.list /etc/apt/sources.list~; \
    sed -Ei 's/^# deb-src /deb-src /' /etc/apt/sources.list; \
    apt-get update; \
    apt-get build-dep -y qt5-default; \
# install compiler cmake and other useful stuff
    apt-get install -y build-essential \
                       cmake \
                       wget \
                       xz-utils \
                       curl \
                       git \
                       dbus \
                       perl \
                       pandoc \
# install opengl libs
                       xorg-dev \
                       libglu1-mesa-dev \
                       freeglut3-dev \
                       libglew1.5 \
                       libglew1.5-dev \
                       libglu1-mesa \
                       libglu1-mesa-dev \
                       libgl1-mesa-glx \
                       libgl1-mesa-dev \
# install qt requirements
                       libxcb-xinerama0-dev \
                       '^libxcb.*-dev' \
                       libx11-xcb-dev \
                       libglu1-mesa-dev \
                       libxrender-dev \
                       libxi-dev \
                       libxkbcommon-dev \
                       libxkbcommon-x11-dev
    
################
# build Python #
################

RUN \
# extract the latest patch release for the given python_version
    wget https://www.python.org/ftp/python/ && python_semvar=`grep -o "${python_version}\.[0-9]*" index.html | sort -V | tail -1` && rm index.html; \
# download and install python
    wget https://www.python.org/ftp/python/${python_semvar}/Python-${python_semvar}.tar.xz; \
    tar xvf Python-${python_semvar}.tar.xz; \
    cd Python-${python_semvar}/; \
    ./configure --enable-optimizations; \
    make -j28 altinstall; \
    cd .. && rm -rf Python-${python_semvar}; \
    update-alternatives --install /usr/bin/python python /usr/local/bin/python${python_version} 1; \
    update-alternatives --install /usr/bin/python3 python3 /usr/local/bin/python${python_version} 1; \
    update-alternatives --set python /usr/local/bin/python${python_version}; \
    update-alternatives --set python3 /usr/local/bin/python${python_version}; \
# Upgrade pip
    curl -s https://bootstrap.pypa.io/get-pip.py -o get-pip.py && \
    python get-pip.py --force-reinstall && \
    rm get-pip.py && \
    pip install -r requirements.txt

#######################################
# build vtk with VTK_WRAP_PYTHON=ON   #
#######################################

# clone vtk
RUN git clone https://github.com/Kitware/VTK.git; \
    cd VTK; \
    git checkout v${vtk_version}; \
# configure build with cmake
    mkdir -p build && cd build ; \
    cmake .. \
    -DBUILD_TESTING=OFF \
    -DCMAKE_BUILD_TYPE=Release \
    -DVTK_WRAP_PYTHON=ON \
    -DVTK_PYTHON_VERSION=3 \
    -DPython3_FIND_STRATEGY=LOCATION; \
# build and install vtk
    make install -j28; \
# cleanup
    cd ../.. && rm -rf VTK

############################################################################
# build qt from source (offline installer not available for FOSS anymore)  #
############################################################################

# clone qt
RUN git clone https://code.qt.io/qt/qt5.git; \
    cd qt5; \
    git checkout v${qt_version}; \
    perl init-repository --module-subset=essential,qtscript; \
# configure qt
    mkdir -p build && cd build; \
    ../configure -opensource -nomake examples -nomake tests -confirm-license; \
# build qt
    make -j28 && make install; \
# cleanup
    cd ../.. && rm -rf qt5


