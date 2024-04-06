SDK version supported: 6.4

<!-- The latest prebuilt release package complete with python bindings and sample applications can be downloaded from the [release section](../../../releases)
for x86 platforms (server). -->

This readme describes how to compile and install DeepStream python bindings (henceforth referred as bindings) for x86 platforms (server). This process is mainly useful for making customizations in the bindings and compiling it yourself instead of using the prebuilt versions provided in the release section.

The readme is divided into three main parts:
- [DeepStream python bindings](#deepstream-python-bindings)
  - [1 Prerequisites](#1-prerequisites)
    - [1.1 Deepstream SDK](#11-deepstream-sdk)
    - [1.2 Base dependencies](#12-base-dependencies)
    - [1.3 Initialization of submodules](#13-initialization-of-submodules)
    - [1.4 Installing Gst-python](#14-installing-gst-python)
  - [2 Compiling the bindings](#2-compiling-the-bindings)
    - [2.1 Quick build (x86-ubuntu-22.04 | python 3.10 | Deepstream 6.4)](#21-quick-build-x86-ubuntu-2204--python-310--deepstream-64)
    - [2.2 Advanced build](#22-advanced-build)
      - [2.2.1 Using Cmake options](#221-using-cmake-options)
      - [2.2.2 Available cmake options](#222-available-cmake-options)
      - [2.2.3 Example](#223-example)
    - [2.3 Cross-Compilation for aarch64 on x86](#23-cross-compilation-for-aarch64-on-x86)
      - [2.3.1 Build Pre-requisites](#231-build-pre-requisites)
      - [2.3.2 Download the JetPack SDK 6.0 DP](#232-download-the-jetpack-sdk-60-dp)
      - [2.3.3 Generate the cross-compile build container](#233-generate-the-cross-compile-build-container)
  - [3 Installing the bindings](#3-installing-the-bindings)
    - [3.1 Installing the pip wheel](#31-installing-the-pip-wheel)
    - [3.2 Launching test-1 app](#32-launching-test-1-app)

<a name="prereqs"></a>
## 1 Prerequisites

The following dependencies need to be met in order to compile bindings:

<a name="prereq_ds"></a>
### 1.1 Deepstream SDK/Deepstream SDK Docker
Go to https://developer.nvidia.com/deepstream-sdk, download and install Deepstream SDK and its dependencies

Docker container
```
docker pull nvcr.io/nvidia/deepstream:6.4-gc-triton-devel

docker pull nvcr.io/nvidia/deepstream:6.4-triton-multiarch

docker pull nvcr.io/nvidia/deepstream:6.4-samples-multiarch
```
Run docker container:
```
# Step to run the docker
export DISPLAY=:0
xhost +
docker run -it --rm --net=host --gpus all -e DISPLAY=$DISPLAY --device /dev/snd -v /tmp/.X11-unix/:/tmp/.X11-unix nvcr.io/nvidia/deepstream:6.4-gc-triton-devel

```

<a name="prereq_base"></a>
### 1.2 Base dependencies
To compile bindings on Ubuntu - 22.04 [use python-3.10, python-3.8 will not work] :
```
apt install python3-gi python3-dev python3-gst-1.0 python-gi-dev git meson \
    python3 python3-pip python3.10-dev cmake g++ build-essential libglib2.0-dev \
    libglib2.0-dev-bin libgstreamer1.0-dev libtool m4 autoconf automake libgirepository1.0-dev libcairo2-dev
```

<a name="prereq_init_sub"></a>
### 1.3 Initialization of submodules
Make sure you clone the deepstream_python_apps repo under <DeepStream ROOT>/sources:
git clone https://github.com/NVIDIA-AI-IOT/deepstream_python_apps

This will create the following directory:
```
<DeepStream ROOT>/sources/deepstream_python_apps
```

The repository utilizes gst-python and pybind11 submodules.
To initializes them, run the following command:
```bash
cd /opt/nvidia/deepstream/deepstream/sources/deepstream_python_apps/
git submodule update --init
```
<a name="prereq_install_gst"></a>
### 1.4 Installing Gst-python

Following commands ensure we add the new certificates that gst-python git server now uses:
```bash
sudo apt-get install -y apt-transport-https ca-certificates -y
sudo update-ca-certificates
```
Update/upgrade the certificates/dependencies
```
apt update
apt upgrade
```
Build and install gst-python:
```bash
cd 3rdparty/gstreamer/subprojects/gst-python/
meson build
meson configure
cd build
ninja
ninja install
```
if `meson build` have an error `pygobject-3.0 meson.build:23:0`
```
sudo apt install python-gi-dev
```


<a name="compile_bindings"></a>
## 2 Compiling the bindings
Python bindings are compiled using CMake.
Following commands provide quick cmake configurations for common compilation options:

<a name="compile_quick"></a>
### 2.1 Quick build (x86-ubuntu-22.04 | python 3.10 | Deepstream 6.4)
```bash
cd deepstream_python_apps/bindings
mkdir build
cd build
cmake ..
make -j$(nproc)
```

<a name="compile_advance"></a>
### 2.2 Advanced build

#### 2.2.1 Using Cmake options

Multiple options can be used with cmake as follows:
```bash
cmake .. [-D<var>=<value> [-D<var>=<value> [-D<var>=<value> ... ]]]
```
#### 2.2.2 Available cmake options

| Var | Default value | Purpose | Available values
|-----|:-------------:|---------|:----------------:
| DS_VERSION | 6.4 | Used to determine default deepstream library path | should match to the deepstream version installed on your computer
| PYTHON_MAJOR_VERSION | 3 | Used to set the python version used for the bindings | 3
| PYTHON_MINOR_VERSION | 10 | Used to set the python version used for the bindings | 10
| PIP_PLATFORM | linux_x86_64 | Used to select the target architecture to compile the bindings | linux_x86_64, linux_aarch64
| DS_PATH | /opt/nvidia/deepstream/deepstream-${DS_VERSION} | Path where deepstream libraries are available | Should match the existing deepstream library folder

#### 2.2.3 Example 

Following commands can be used to compile the bindings natively on dGPUnm devices

```bash
cd deepstream_python_apps/bindings
mkdir build
cd build
cmake ..  -DPYTHON_MAJOR_VERSION=3 -DPYTHON_MINOR_VERSION=10 \
    -DPIP_PLATFORM=linux_x86_64 -DDS_PATH=/opt/nvidia/deepstream/deepstream/
make
```


Build output (pip wheel) is copied to the previously created export_pyds directory (deepstream_python_apps/bindings/export_pyds) on the host machine.

<a name="install_bindings"></a>
## 3 Installing the bindings
Following commands can be used to install the generated pip wheel.

<a name="install_wheel"></a>
### 3.1 Installing the pip wheel
```bash
pip3 install ./pyds-1.1.10-py3-none*.whl
```
If the wheel installation fails, upgrade the pip using the following command:
```bash
python3 -m pip install --upgrade pip
```

<a name="install_launch"></a>
### 3.2 Launching test-1 app
```bash
cd apps/deepstream-test1
python3 deepstream_test_1.py <input .h264 file>

# e.g.
python3 deepstream_test_1.py file:///opt/nvidia/deepstream/deepstream/samples/streams/sample_720p.mp4                                                         
```
if have an issue `Authrization required`. Run this command outside of the docker container environment
```
export DISPLAY=:0
xhost +
```
