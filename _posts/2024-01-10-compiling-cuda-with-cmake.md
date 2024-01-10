---
layout: post
title:  "Compiling CUDA Projects With CMake"
date:   2023-12-14 10:30:00 -0600
categories: cmake
modified_date:   2024-01-10 11:00:00 +0000
---

Often times the guides for compiling .cu files involve building everything with `nvcc` command, which is fine for proof of concepts, but for production-like projects, where dependencies and header files are involved, compiling everything with `nvcc` command gets trickier. For better dependency experience in your c++ projects, CMake can also handle the CUDA building pipeline. 

This guide will hopefully help anyone who's trying to create a larger CUDA project and make it out alive. The project that tested this guide has the following structure:

```
cuda/
├─ qrcode/
|   ├─ QrCode.cu
|   ├─ CMakeLists.txt
├─ main.cu
├─ CMakeLists.txt
```

The main file `main.cu` contains the `__globaḷ__` kernel function that uses a struct defined in `qrcode` folder. Therefore, `main.cu` includes `qrcode/QrCode.cu` file as `#include <./qrcode/QrCode.cu>`.

## CMake Config Files

There must be a `CMakeLists.txt` file in the root folder of the directory and inside every subfolder of the project. In this case, we got two `CMakeLists.txt` files with the following:

- `./CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.8 FATAL_ERROR) # must be 3.18 or higher
project(main LANGUAGES CXX CUDA) # include c++ and cuda
set(CMAKE_CUDA_ARCHITECTURES 50) # the '50' is the computing capability of the gpu

# installed packages in your system
find_package(CUDA REQUIRED)
find_package(ZXing REQUIRED) # qrcode external library
find_package(OpenCV REQUIRED) # opencv (4.8.0)

add_subdirectory(qrcode) # the subfolder

include_directories(${OpenCV_INCLUDE_DIRS})

# order of commands is key here:
add_executable(main main.cu)
target_sources(main PRIVATE qrcode/QrCode.cu) # link the sources
target_link_libraries(main ${OpenCV_LIBS} ZXing::ZXing) # link external libs
target_include_directories(main PRIVATE /usr/local/include)

SET (CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -arch=sm_50 -std=c++17") # cuda flags, here '50' also refers to the computing capability of the gpu
```

- `./qrcode/CMakeLists.txt`:

```cmake
add_library(qrcode OBJECT QrCode.cu)

set_target_properties(qrcode PROPERTIES
    CUDA_SEPARABLE_COMPILATION ON
)
```

## Compilation Process

As seen in the previous section, the current project makes use of two external libraries: ZXing and OpenCV. The installation must be performed as if it was c++ code. The guide for that can be found in this site.

1. Clone, build and install ZXing and OpenCV. Recomended to install OpenCV first so that ZXing can find it at the time of its installation and activate the OpenCV module.

2. Install `cmake`:

```
sudo apt-get install cmake
```

3. To confirm installation, type:

```
cmake --version
```

**Important**: if the version displayed is older or less than `3.18`, you will likely get an error saying `CUDA_STANDARD is set to invalid value '17'` when you perform the command `cmake .`. This is explained in [this Github Issue](https://github.com/getkeops/keops/issues/122), but basically, cmake versions before 3.18 don't include the CUDA Standard value of 17, and this value is the one that sets as default by the CUDA package in your system.

To remove cmake if your installation is older than version 3.18, follow the steps:

4. Remove cmake:

```
sudo apt-get remove --purge cmake
```

5. Download and execute the installer for version 3.18:

```
wget https://cmake.org/files/v3.18/cmake-3.18.0-Linux-x86_64.sh
chmod +x cmake-3.18.0-Linux-x86_64.sh
sudo ./cmake-3.18.0-Linux-x86_64.sh --skip-license --prefix=/usr
```

6. Now, you can finally build and run CUDA projects:

```
cmake .
make
./main
```


## Notes

This guide was used for compilation using the following GPUs:

- Nvidia Tesla V100 16GB: uses computing capability of `70`.

- Nvidia GTX 460: uses computing capability of `50`.

The only thing that changes is **every command that refers to the computing capability**. Other than that, this quide works for any different Nvidia GPU.