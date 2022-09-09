---
processors: ["Neoverse-N1"]
software: ["linux"]
title: "Install and build Vectorscan on Arm-based servers"
linkTitle: "Build Vectorscan on Arm"
type: docs
weight: 1
hide_summary: true
description: >
    Learn how to build Vectorscan, a regex parsing library based on Intel's Hyperscan library on AWS EC2 instances powered by Arm64 achitecture.
---

## Pre-requisites

* An [Arm based instance](/cloud/platforms) from an appropriate cloud service provider
* Vectorscan is known to work on the following Linux distributions. 
   * RHEL/CentOS 8
   * Ubuntu Versions - 22.04, 20.04, 18.04

The instructions provided below have been tested on an Ubuntu 22.04 AWS 64-bit Arm EC2 instance (C6g.xlarge) and Ubuntu 20.04 Oracle Ampere A1 instance.

* GCC for your Arm Linux distribution. Install using the steps [here](/compilers/install_ngcc).
* [cmake](https://cmake.org/) - used here as the build system:
```console
sudo apt install cmake
```
* [Boost](https://www.boost.org/) C++ libraries
```console
sudo apt install libboost-all-dev
```
* [Ragel](https://packages.ubuntu.com/bionic/ragel)
```console
sudo apt install ragel
```
* [PkgConfig](https://en.wikipedia.org/wiki/Pkg-config)
```console
sudo apt install pkg-config
```
* [Sqlite3](https://www.sqlite.org/index.html)
```console
sudo apt install libsqlite3-dev
```
* [libpcap](https://www.tcpdump.org/) - Package for capturing network packets
```console
sudo apt install libpcap-dev
```

## Detailed Steps

[Vectorscan](https://github.com/VectorCamp/vectorscan) is an architecture-inclusive fork of [Hyperscan](https://github.com/intel/hyperscan), that preserves the support for x86 and modifies the framework to allow for Arm architectures and vector engine implementations.

### Install Vectorscan

Start by cloning the git repository for Vectorscan.
```console
git clone https://github.com/VectorCamp/vectorscan.git
cd vectorscan
```

### Edit environment variables and fix PCRE download location

You must first fix the [PCRE](https://www.pcre.org/) download location in the `cmake` file. Open `cmake/setenv-arm64-cross.sh` in an editor of your choice, for example:

```console
vi cmake/setenv-arm64-cross.sh
```
Change `https://ftp.pcre.org/pub/pcre/pcre-8.41.tar.bz2` to `https://sourceforge.net/projects/pcre/files/pcre/8.41/pcre-8.41.tar.bz2/download`.

In the same file, now set the environment variables to match the location of your `GCC compiler`, `Boost libraries` and `PCRE` installation.

The snippet below is an example with the default installation locations.
```console
export BOOST_VERSION=1_57_0
export BOOST_DOT_VERSION=${BOOST_VERSION//_/.}
export CROSS=/usr/bin/aarch64-linux-gnu-
export CROSS_SYS=/

# if [ ! -d "boost_$BOOST_VERSION" ];
# then
#       wget -O boost_$BOOST_VERSION.tar.gz https://sourceforge.net/projects/boost/files/boost/$BOOST_DOT_VERSION/boost_$BOOST_VERSION.tar.gz/download
#       tar xf boost_$BOOST_VERSION.tar.gz
# fi
if [ ! -d "pcre-8.41" ];
then
        wget -O pcre-8.41.tar.bz2 https://sourceforge.net/projects/pcre/files/pcre/8.41/pcre-8.41.tar.bz2/download
        tar xf pcre-8.41.tar.bz2
        export PCRE_SOURCE=./pcre-8.41
fi

export BOOST_PATH=/usr/include
```
After closing and saving, `source` this file:
```console
source cmake/setenv-arm64-cross.sh
```

### Fix source to build with glibc>=2.34

There is a current issue where builds fail with `glibc >= 2.34` and a pending [PR](https://github.com/intel/hyperscan/issues/359).

For now, workaround this issue by making the changes to `STACK_SIZE` as mentioned in the [pull request](https://github.com/intel/hyperscan/pull/358/files/eac1e5e0354f3ead2c832e798d89f86082b77d75).

### Configure Vectorscan with cmake

Create build directory and configure cmake to build vectorscan. 

```console
mkdir vectorscan-build
cd vectorscan-build
cmake -DCROSS_COMPILE_AARCH64=1 ../ -DCMAKE_TOOLCHAIN_FILE=../cmake/arm64-cross.cmake
```

To build vectorscan on targets with [Scalable Vector Extensions (SVE)](https://developer.arm.com/Architectures/Scalable%20Vector%20Extensions) support, define one of the below variables on your `cmake` command (only one of these variables needs to be set as weaker variables will be implied as set).

* `BUILD_SVE`
* `BUILD_SVE2`
* `BUILD_SVE2_BITPERM`

For example:

```console 
cmake -DCROSS_COMPILE_AARCH64=1 ../ -DCMAKE_TOOLCHAIN_FILE=../cmake/arm64-cross.cmake -DBUILD_SVE=1
```

### Build Vectorscan 

In the final step, use `make` to build the vectorscan library

```console
make -jT
```

Where `T` specifies the number of threads used to build.

The executables from the build will be created in the `bin` directory.

### Run Vectorscan Unit Tests

Run a check to validate that `Vectorscan` is built and running correctly:

```console
./bin/unit-hyperscan
```

All the unit tests should run successfully. At the end of execution you will see output similar to:

```
[----------] Global test environment tear-down
[==========] 3746 tests from 33 test cases ran. (197558 ms total)
[  PASSED  ] 3746 tests.
```

[<-- Return to Learning Path](/cloud/vectorscan/#sections)
