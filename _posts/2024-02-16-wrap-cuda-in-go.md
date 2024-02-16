---
layout: post
title:  "Wrap A CUDA Application In Golang"
date:   2024-02-16 09:10:00 -0600
categories: cuda
modified_date:   2024-02-16 10:20:00 +0000
--- 

In todays standards, applications very rarely stay as standalone, and instead the usual step further is to give the user an access point through the internet, through an API. 

Once you got a C/C++ app that uses a kernel with CUDA, how can we make it accessible through an API? We could code a server in C++, but is that the most maintainable option?

Personally, I prefer to use a language that is more suited for internet access endpoints. Due to some other requirements I will not mention, I chose Go. But can we call an **executable** in Go? The answer is yes. It can be achieved using CGo.

## Create A Shared Library

In this text I assume you got a CMake build pipeline to create an **executable file for your project**. For the wrapping, we need to modify the CMake files in order to ditch the executable and create a **shared library** instead. In Linux, a shared library has the .so extension, while in Windows it stands as .dll. Most servers are cheaper if you run a Linux machine, so we will go on with the Linux way.

CGo calls **C** code, but since CUDA is compiled with C++ syntax, we need to basically create a C interface that can call C++ code. CGo calls this interface, and this interface calls C++/CUDA code.

### Adjust The Code

The project that we will wrap has the following structure:

```
cuda/
├─ myfolder1/
|   ├─ MyFile1.cu
|   ├─ CMakeLists.txt
├─ myfolder2/
|   ├─ MyFile2.cu
|   ├─ CMakeLists.txt
├─ mylib1/
|   ├─ Lib1.h
|   ├─ Lib1.cpp
|   ├─ CMakeLists.txt
├─ main.cu
├─ CMakeLists.txt
```

- `main.cu`

```c++
// external dependencies
#include <cuda_runtime.h>
#include <ZXing/BarcodeFormat.h>
#include <opencv2/opencv.hpp>

// internal dependencies
#include "myfolder1/MyFile1.cu"
#include "myfolder2/MyFile2.cu"
#include "mylib1/Lib1.h"

using namespace cv;
using namespace ZXing;
__global__ myKernel(){
    // defined in MyFile1.cu
    MyClass1 obj1();
    obj1.someFunc();
    // defined in MyFile2.cu
    MyClass2 obj2();
    obj2.someFunc();
}

void generate() {
    // defined in Lib1.h
    MyClass::myFunc();
    myKernel<<<1, 10>>>();
}

int main() {
    generate();
}
```

It's important to note that main.cu depends on MyFile1.cu and MyFile2.cu to work, but also on OpenCV and ZXing (C++ libraries). The function we want to be able to call from Go is `generate()`.

1. The first thing to do is to create a header file for `main.cu`, for example `main_interface.hpp`. It's important to use the `.hpp` extension so that it's compiled with the C++ compiler, since it's a header file for CUDA code, which uses C++ syntax.

```
cuda/
├─ myfolder1/
|   ├─ MyFile1.cu
|   ├─ CMakeLists.txt
├─ myfolder2/
|   ├─ MyFile2.cu
|   ├─ CMakeLists.txt
├─ mylib1/
|   ├─ Lib1.h
|   ├─ Lib1.cpp
|   ├─ CMakeLists.txt
├─ main.cu
├─ main_interface.h <-----
├─ CMakeLists.txt
```

2. Include the `extern "C"` directive in the function declaration between preprocessor directives to ensure it's recognized only by C++ compiler. The `extern` linkage especifier is not recognized by C, so that's why we enclose it with preprocessor directives:

- `main_interface.h`

```c++
#ifndef MAIN_INTERFACE_H
#define MAIN_INTERFACE_H

#ifdef __cplusplus
extern "C" {
#endif

void generate();

#ifdef __cplusplus
}
#endif

#endif
```

3. Since `main.cu` now is the implementation of `main_interface.hpp` header file, we need to include the header in `main.cu` and enclose `generate()` with `extern "C"` linkage specifier. This time, we don't need preprocessor directives like `#ifdef` used above because we know that `main.cu` will be compiled by nvcc which recognizes C++ syntax.

- `main.cu`

```c++
// external dependencies
#include <cuda_runtime.h>
#include <ZXing/BarcodeFormat.h>
#include <opencv2/opencv.hpp>

// internal dependencies
#include "myfolder1/MyFile1.cu"
#include "myfolder2/MyFile2.cu"
#include "main_interface.hpp" // <----- add this

using namespace cv;
using namespace ZXing;
__global__ myKernel(){
    // defined in MyFile1.cu
    MyClass1 obj1();
    obj1.someFunc();
    // defined in MyFile2.cu
    MyClass2 obj2();
    obj2.someFunc();
}

extern "C" {
    void generate() {
        myKernel<<<1, 10>>>();
    }
}

int main() {
    generate();
}
```

The `extern "C"` makes the compiler not mangle its function name, which is often done in C++ compilation, and by doing this we make `generate()` **visible to C code**.

4. Create a wrapper folder with some C files in the folder where `main.cu` is.

```
mkdir wrapper
touch wrapper/Wrapper.c
touch wrapper/Wrapper.h
touch wrapper/CMakeLists.txt
```

and thus, the project will look like this:

```
cuda/
├─ myfolder1/
|   ├─ MyFile1.cu
|   ├─ CMakeLists.txt
├─ myfolder2/
|   ├─ MyFile2.cu
|   ├─ CMakeLists.txt
├─ mylib1/
|   ├─ Lib1.h
|   ├─ Lib1.cpp
|   ├─ CMakeLists.txt
├─ wrapper/
|   ├─ Wrapper.c
|   ├─ Wrapper.h
|   ├─ CMakeLists.txt
├─ main.cu
├─ main_interface.h
├─ CMakeLists.txt
```

This folder will contain the C interface that we will include in our Go code.

5. Provide the C code necessary for it to be a library that *calls our CUDA function* `generate()`.

- `wrapper/Wrapper.h`

```c
#ifndef WRAPPER_H
#define WRAPPER_H

void WrapGenerate();

#endif
```

- `wrapper/Wrapper.c`

```c
#include "../main_interface.hpp"
#include "Wrapper.h" 

void WrapGenerate() {
	generate();
}
```

In this way, `WrapGenerate()`, which is visible to CGo, calls the function `generate()`, declared in `../main_interface.hpp` but implemented in `main.cu`.

### Adjust CMake

As mentioned before, we need to stop generating an **executable file** (.o) and generate a **shared library** (.so) instead. For this, we need to modify our CMake files.

1. Add `add_library` with the key word `SHARED` instead of `add_executable` in the `wrapper/CMakeLists.txt`:

- `wrapper/CMakeLists.txt`:
```cmake
cmake_minimum_required(VERSION 3.8 FATAL_ERROR)
project(wrapper LANGUAGES C CXX CUDA)  # include C, C++ and CUDA langs

find_package(CUDA REQUIRED)

# include here all files that main.cu depends on too
add_library(wrapper SHARED
    ${CMAKE_SOURCE_DIR}/main_interface.hpp
    ${CMAKE_SOURCE_DIR}/main.cu
    ${CMAKE_SOURCE_DIR}/myfolder1/MyFile1.cu
    ${CMAKE_SOURCE_DIR}/myfolder2/MyFile2.cu
    ${CMAKE_SOURCE_DIR}/wrapper/Wrapper.c
    ${CMAKE_SOURCE_DIR}/wrapper/Wrapper.h
)
set_target_properties(wrapper PROPERTIES ENABLE_EXPORTS 1)
target_include_directories(wrapper PUBLIC ${CMAKE_SOURCE_DIR})

# include here all libraries (internal or external) that main.cu depends on
target_link_libraries(wrapper PRIVATE ${OpenCV_LIBS} ZXing::ZXing mylib1)
```

Here, it's crucial to include in the `add_library` command all files that `main.cu` depends on, or else the shared library will not include `generate()` because its implementation's dependencies are unknown.

- `./CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.8 FATAL_ERROR)
project(main LANGUAGES C CXX CUDA)
set(CMAKE_CUDA_ARCHITECTURES 70)

find_package(CUDA REQUIRED)
find_package(ZXing REQUIRED)
find_package(OpenCV REQUIRED)

add_subdirectory(myfolder1)
add_subdirectory(myfolder2)
add_subdirectory(mylib1)
add_subdirectory(wrapper)

include_directories(${OpenCV_INCLUDE_DIRS})
include_directories(${CMAKE_SOURCE_DIR}/wrapper)

# all files that main.cu uses
add_library(main SHARED 
    main_interface.hpp
    main.cu
    myfolder1/MyFile1.cu
    myfolder2/MyFile2.cu
    wrapper/Wrapper.c
)

# libraries that main.cu uses
target_link_libraries(main ${OpenCV_LIBS} ZXing::ZXing mylib1)
target_sources(main PRIVATE wrapper)
set_property(TARGET mylib1 PROPERTY POSITION_INDEPENDENT_CODE ON)

SET (CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -arch=sm_70 -std=c++17")
```

And, just to leave an example of a CMake file to generate `mylib1` as a separated internal library (for aws calls, for example):

- `mylib1/CMakeLists.txt`:

```cmake
add_library(mylib1 Lib1.cpp Lib1.h)

# external libs
target_link_libraries(mylib1 PRIVATE ${OpenCV_LIBS})
```

2. Now we can generate the shared libraries. Go to the root of the project and type:

```
cmake .
make
```

This will generate two shared libraries:

```
cuda/
├─ myfolder1/
|   ├─ MyFile1.cu
|   ├─ CMakeLists.txt
├─ myfolder2/
|   ├─ MyFile2.cu
|   ├─ CMakeLists.txt
├─ mylib1/
|   ├─ Lib1.h
|   ├─ Lib1.cpp
|   ├─ CMakeLists.txt
├─ wrapper/
|   ├─ Wrapper.c
|   ├─ Wrapper.h
|   ├─ libwrapper.so <-----
|   ├─ CMakeLists.txt
├─ main.cu
├─ main_interface.h
├─ libmain.so <-----
├─ CMakeLists.txt
```

The one we need to use in Go is `wrapper/libwrapper.so`. CGo can compile C projects, but since ours has CUDA/C++, compiling it with Go would be a pain. Therefore, we compile the shared libs first using CMake, and then we link the necessary .so file to our Go code. 

CGo will look for `generate()` in the symbol table inside our .so library that we tell it to use. In order to make sure that the symbol for `generate()` is present, we can run the command:

```
nm -D wrapper/libwrapper.so | grep generate
```

and the output, if found, must be similar to this:

```
0000000000017584 T generate
```

where the `T` tells us that this symbol is indeed extracted from the text in the .so file. Something else means that the symbol is not found, and therefore CGo won't find it. Tweak your CMakeLists.txt file until you find the symbol.

Since the symbol is found, we can proceed to our Go code.

## Create A Go Project

1. Create a folder in the same level as the `cuda/` folder:

```
├─ cuda/
|   ├─ myfolder1/
|   ├─ myfolder2/
|   ├─ mylib1/
|   ├─ wrapper/
├─ go/
|   ├─ main.go
```

2. Add the necessary Go code so that our `libwrapper.so` is linked, not compiled, by CGo.

```go
package main

// #cgo CFLAGS: -I../cuda/wrapper
// #cgo LDFLAGS: -L../cuda/wrapper -lwrapper
// #include "Wrapper.h"
import "C"

func main() {
	C.WrapGenerate()
}

```

The comments here are mandatory, since they are CGo directives. The `-I` flag determines the include directory where the `Wrapper.h` can be found. The `-L` determines the directory where the `libwrapper.so` can be found. The `-lwrapper` flag must have the structure `-l` plus the name of your .so file without the extension and the word `lib`.

3. Compile your go app like this:

```
LD_LIBRARY_PATH=/home/Documents/my-project/cuda/wrapper go build
```

4. Run the executable like this:

```
LD_LIBRARY_PATH=/home/Documents/my-project/cuda/wrapper ./wrapper
```

We specify `LD_LIBRARY_PATH` variable with the path where the `libwrapper.so` is located because the shared lib is not installed in any standard install location, such as `/usr/local` or `/usr/lib`, and so we need to tell the go-tool where the library is located.

When you run the go executable, you will now see your CUDA app output, if you got any, alongside your go output.