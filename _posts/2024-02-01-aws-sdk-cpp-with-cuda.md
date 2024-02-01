---
layout: post
title:  "Integrating AWS SDK C++ With CUDA"
date:   2024-02-01 09:10:00 -0600
categories: cuda
modified_date:   2024-01-10 12:20:00 +0000
---

Even though CUDA uses C++ syntax, most C++ libraries include features that are not allowed in CUDA. All the libraries you want to use in your CUDA project must be used in C++ code. Usually, as long as your library is called from the host code, it will work just fine. However, there are some libraries, such as AWS SDK, that cannot be compiled by CUDA at all. This guide is made for the purpose of creating such a separation in the CMake config of the project so that AWS SDK is not accessed by any CUDA compiler.

For this guide, image you got a project with the structure:

```
cuda/
├─ matrix_factory/
|   ├─ MatrixFactory.h
|   ├─ MatrixFactory.cpp
|   ├─ CMakeLists.txt
├─ main.cpp
├─ CMakeLists.txt
├─ myfolder1/
|   ├─ myfile.cu
|   ├─ CMakeLists.txt
```

## Install The SDK

1. Go to the root of your GPU machine and clone the AWS SDK for C++:

```
git clone --recurse-submodules https://github.com/aws/aws-sdk-cpp
```

2. Enter the cloned repo:

```
cd aws-sdk-cpp
```

3. Make a build directory and enter:

```
mkdir build
cd build
```

4. Make a directoro for the system installation:

```
sudo mkdir /usr/lib/aws-sdk-cpp
```

5. Remember we are inside `aws-sdk-cpp/build`, so generate the build files:

```
cmake .. -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_PREFIX=/usr/lib/aws-sdk-cpp -DBUILD_ONLY="s3"
```

6. Build it

```
cmake --build . --config=Debug
```

7. Install it

```
sudo cmake --install . --config=Debug
```

## Integrate It Using CMake

1. Create a separate folder, in this case, `matrix_factory`. We will create a header file and the implementation file so that there is even more emphasizing of the separation from the CUDA code.

```
mkdir matrix_factory
touch matrix_factory/MatrixFactory.cpp
touch matrix_factory/MatrixFactory.h
```

2. In the header file, include the class definition and the includes:

`.h file`
```c++
#pragma once

// other includes here ...

// aws-related includes here ...
#ifndef __CUDACC__
#include <aws/core/Aws.h>
#include <aws/s3/S3Client.h>
#include <iostream>
#include <aws/core/auth/AWSCredentialsProviderChain.h>

using namespace Aws;
using namespace Aws::Auth;
#endif

// other namespaces here ...
class MatrixFactory {
	public:
		static void aws(); // function that uses sdk	
};
```

Here the `#ifndef __CUDACC__` include condition ensures these AWS SDK includes and namespaces are **only present when compiling with a C++ compiler**, since `__CUDACC__` is not defined when compiling CUDA code.

3. In the implementation file (.cpp), implement the function:

```c++
#include "./MatrixFactory.h"
using namespace Aws;
using namespace Aws::Auth;
void MatrixFactory::aws() {
    Aws::SDKOptions options;
	Aws::InitAPI(options);
}
```

4. Then, this will allow us to **remove all inclusions of the AWS SDK in the root CMakeLists.txt**, since this file handles CUDA compiling for `main.cu`. Modify your `./CMakeLists.txt` and `matrix_factory/CMakeLists.txt` as follows:

- `./CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.8 FATAL_ERROR)
project(main LANGUAGES CXX CUDA)
set(CMAKE_CUDA_ARCHITECTURES 70)
set(CMAKE_PREFIX_PATH "/usr/lib/aws-sdk-cpp") # installation path

find_package(CUDA REQUIRED)
# more packages here ...
find_package(ZXing REQUIRED)
find_package(OpenCV REQUIRED)
find_package(AWSSDK COMPONENTS s3 REQUIRED) # this is allowed

add_subdirectory(myfolder1)
add_subdirectory(matrix_factory) # include the cpp module lib here

include_directories(${OpenCV_INCLUDE_DIRS})
# DO NOT include_directories(${AWSSDK_INCLUDE_DIRS})

add_executable(main main.cu 
	myfolder1/myfile1.cu 
    # more files here ...
    # DO NOT INCLUDE matrix_factory/MatrixFactory.cpp HERE
)

target_link_libraries(main 
    ${OpenCV_LIBS} 
    ZXing::ZXing
    # more package includes here ...
    matrix_factory # the cpp module lib
    ${AWSSDK_LINK_LIBRARIES} # this is allowed
) 

# your cuda flags here ...
SET (CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -arch=sm_70 -std=c++17")
```

- `matrix_factory/CMakeLists.txt`

```cmake
add_library(matrix_factory MatrixFactory.cpp MatrixFactory.h)

target_link_libraries(matrix_factory PRIVATE ZXing::ZXing)
# more linking for libs that this module uses here ...
# add these two lines
target_include_directories(matrix_factory PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
target_include_directories(matrix_factory PRIVATE ${AWSSDK_INCLUDE_DIRS})

# 
target_compile_definitions(matrix_factory PRIVATE -DUSE_IMPORT_EXPORT)

target_link_libraries(matrix_factory PRIVATE aws-cpp-sdk-core aws-cpp-sdk-s3)
```

Here:

- `target_compile_definitions(matrix_factory PRIVATE -DUSE_IMPORT_EXPORT)`: This command defines the preprocessor macro `USE_IMPORT_EXPORT` for the `matrix_factory` target. The `-D` flag is used to define a preprocessor macro, and `PRIVATE` specifies that the definition applies only to the target specified (`matrix_factory`).

The `USE_IMPORT_EXPORT` macro itself doesn't have a standard meaning in CMake; it's a custom macro that you can use in your code as a conditional compilation flag. For example, in your code, you might have conditional compilation blocks like:

```c++
Copy code
#ifdef USE_IMPORT_EXPORT
// Code related to the USE_IMPORT_EXPORT feature
#endif
```

This allows you to selectively include or exclude certain parts of the code based on whether the `USE_IMPORT_EXPORT` macro is defined during compilation.

- `target_link_libraries(matrix_factory PRIVATE aws-cpp-sdk-core aws-cpp-sdk-s3)`: 
The `target_link_libraries` CMake command is used to specify libraries or targets that a given target (in this case, `matrix_factory`) depends on. This line adds two dependencies (`aws-cpp-sdk-core` and `aws-cpp-sdk-s3`) to the matrix_factory target. It tells CMake that the matrix_factory target requires the libraries `aws-cpp-sdk-core` and `aws-cpp-sdk-s3` to be linked during the linking phase.

5. Now you can go to `main.cu` and include **only the header file**:

```c++
#include "matrix_factory/MatrixFactory.h"
int main(){
    MatrixFactory::aws();
}
```

and compile a CUDA project that uses AWS SDK:

```
cmake .
make
./main
```