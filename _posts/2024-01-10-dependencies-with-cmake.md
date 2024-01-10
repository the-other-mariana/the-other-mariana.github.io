---
layout: post
title:  "C++ Dependencies With CMake"
date:   2024-01-10 09:10:00 -0600
categories: cmake
modified_date:   2024-01-10 10:20:00 +0000
---

CMake is a great tool to manage your dependencies smoothly in C++ projects, but some open source repositories don't provide installation info for CMake projects. This guide aims to show you how to include repositories in your projects using CMake. In particular, ZXing (famous QR code library) and OpenCV (famous computer vision library).

*Note: maybe it is a good idea to install OpenCV before ZXing, since ZXing enables an extra module if it finds OpenCV in the system.

## Example 1: Include ZXing

1. Go to your c++ project directory, for example mine looks like this:

```
cuda/
├─ matrix/
|   ├─ Matrix.h
|   ├─ Matrix.cpp
|   ├─ CMakeLists.txt
├─ main.cpp
├─ CMakeLists.txt
```

```
cd cuda
```

2. Clone the ZXing repo (inside your project, for better order):

```
git clone git@github.com:nu-book/zxing-cpp.git
```

3. Make a folder for it inside `/usr/lib/`:

```
sudo mkdir /usr/lib/zxing-cpp/
```

4. Build the package in the folder we made (you must be located still in the root directory of the project, so that folder `./zxing-cpp` is visible):

```
sudo cmake -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local -G "Unix Makefiles" -S zxing-cpp/ -B /usr/lib/zxing-cpp/
```

With these command we are setting:

- `-D CMAKE_INSTALL_PREFIX=/usr/local`: Specifies the installation prefix where the ZXing library will be installed. In this case, you've set it to the system-wide location `/usr/local`, which is a common location for installing libraries and headers that are meant to be available system-wide.

- `-S zxing-cpp/`: Sets the source directory where CMake should look for the project's CMakeLists.txt file. The source code for the ZXing library is located in the `zxing-cpp/` directory. In this case, CMake now understands that the source code is inside `cuda/zxing-cpp`.

- `-B /usr/lib/zxing-cpp/`: Sets the build directory where CMake generates build files, such as Makefiles, build scripts, and other intermediate files. In this case, you've set it to `/usr/lib/zxing-cpp/`.

    - If you go to `/usr/lib/zxing-cpp/` and see what's in there: 

    ```
    CMakeFiles  cmake_install.cmake  libZXing.so  libZXing.so.2.1.0  libZXing.so.3  Makefile  ZXingConfig.cmake  ZXingConfigVersion.cmake  zxing.pc  ZXVersion.h
    ```

    - Here, CMake generates all the build-related files and folders required for building the ZXing library. The build process compiles the source code and creates the library files and CMake configuration files.

5. Go to the folder:

```
cd /usr/lib/zxing-cpp/
```

6. Install the package:

```
sudo make install
```

- After running `sudo make install`, the ZXing library files and CMake configuration files are installed in system-wide locations such as `/usr/local/include` and `/usr/local/lib/cmake`.

    - If we go to `/usr/local/include/ZXing`, you will find all header files that are available to `#include` in your cpp files:

    ```
    BarcodeFormat.h  BitMatrix.h    ByteArray.h     Content.h      Error.h  GTIN.h       Matrix.h             Point.h          Range.h        Result.h            TextUtfEncoding.h  ZXConfig.h
    ...
    ```

    - If we go to `/usr/local/lib/cmake/ZXing`, you will find the CMake config files as well, that the line `find_package(ZXing REQUIRED)` is looking for:

    ```
    ZXingConfig.cmake  ZXingConfigVersion.cmake  ZXingTargets.cmake  ZXingTargets-release.cmake
    ```

    - If we go to `/usr/lib/zxing-cpp/core/CMakeFiles/Export/lib/cmake/ZXing`, we will find also CMake related files:

    ```
    ZXingTargets.cmake  ZXingTargets-release.cmake
    ```

7. Add to your project's CMakeLists.txt at `./CMakeLists.txt`:

```cmake
find_package(ZXing REQUIRED PATHS /usr/lib/zxing-cpp/core)
cmake_policy(SET CMP0048 NEW)
cmake_minimum_required(VERSION 3.15)
project( main )

find_package(ZXing REQUIRED PATHS /usr/lib/zxing-cpp/core) # added, but it can be just find_package(ZXing REQUIRED) also
find_package( OpenCV REQUIRED )
include_directories( ${OpenCV_INCLUDE_DIRS} )
add_subdirectory(qrcode)

add_executable( main main.cpp )
target_link_libraries( main ${OpenCV_LIBS} ZXing::ZXing qrcode) # added
target_include_directories(main PRIVATE /usr/local/include) # addes
```

- `find_package` command searches for package configuration files in a list of well-known locations, including directories specified by the `CMAKE_PREFIX_PATH` variable. The `CMAKE_PREFIX_PATH` variable is used to provide additional hints about where to find CMake packages.

- You installed the ZXing library using the CMAKE_INSTALL_PREFIX option, which set the installation prefix to `/usr/local`. CMake automatically looks in the system-wide installation directories, including `/usr/local/lib/cmake/ZXing`, to find the package configuration files (e.g., ZXingConfig.cmake).

- The use of `find_package(ZXing REQUIRED)` should ideally be sufficient to locate the ZXing library and its package configuration files, assuming that the installation was performed correctly.

- The alternative `find_package(ZXing REQUIRED PATHS /usr/lib/zxing-cpp/core)` is a specific hint to CMake to look in a particular directory (in this case, `/usr/lib/zxing-cpp/core`) for the package configuration files. While this approach can work, it's generally not the standard way to use `find_package`, and it might be less portable across different systems.

8. If you check the header files inside `/usr/local/lib/cmake/ZXing` (step 6), you can now include in you cpp code the library:

```cpp
#include <ZXing/BarcodeFormat.h>

int main(){

}
```

- It's in the format `<ZXing/...>` because of the installation path `/usr/local/include/ZXing/`.

Sources: https://stackoverflow.com/questions/66059595/building-zxing-zebra-crossing-library-on-linux

## Example 2: Include OpenCV Full

1. Install the needed modules:

```
cd ~
sudo apt install -y g++ cmake make git libgtk2.0-dev pkg-config
```

2. Clone opencv classic (still in ~):

```
git clone https://github.com/opencv/opencv.git
mkdir -p build
```

3. Clone extra opencv modules:

```
git clone https://github.com/opencv/opencv_contrib
```

4. Build both:

```
cmake -DOPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules ~/opencv
make -j4
```

5. Install them system-wide:

```
sudo make install
```

- This command will copy header files to `/usr/local/include/opencv2`:

    ```
    ccalib.hpp        dpm.hpp         fuzzy.hpp      img_hash.hpp
    ...
    ```

- CMake configuration files for OpenCV and its extra modules are stored in `/usr/local/lib/cmake/opencv`. These files are used by CMake when configuring projects that depend on OpenCV.

    ```
    libopencv_dpm.so                 libopencv_imgproc.so                    libopencv_rapid.so                   libopencv_tracking.so
    ...
    ```

6. Now, in your project's `./CMakeLists.txt`, add these:

```cmake
find_package( OpenCV REQUIRED )
include_directories( ${OpenCV_INCLUDE_DIRS} )
...
target_link_libraries( main ${OpenCV_LIBS} ...)
```

- When you execute `find_package(OpenCV REQUIRED)` in your CMake project, CMake searches for the OpenCV CMake configuration files in standard locations, including system-wide installation paths like` /usr/local/lib/cmake/opencv`.

    ```
    OpenCVConfig.cmake  OpenCVConfig-version.cmake  OpenCVModules.cmake  OpenCVModules-release.cmake
    ```

- Therefore, as long as your desired lib has its CMakeConfig files inside a directory in `/usr/local/lib/cmake`, your `find_package()` should work.

## Conclusion

- When installing a c++ dependency using CMake, make sure to:

    - clone the repo

    - build the source

    - install the built (compiled) source: this step will make some CMake files in `/usr/local/`, in some `cmake` sub folder.

    - add the lines `find_package()` in the project's `./CMakeLists.txt` so that the package is found.

    - `#include` the lib's header files now, find them at `/usr/local/include/your-lib-name`.
