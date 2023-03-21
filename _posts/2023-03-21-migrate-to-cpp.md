---
layout: post
title:  "Migrate To C++"
date:   2023-03-21 09:30:00 -0600
categories: qr code embedding
modified_date:   2023-03-21 11:30:00 +0000
---

## Build a C++ OpenCV System with CMake

Cmake is cross-platform, open-source build system for managing the build process of software using a compiler-independent method. In most cases it is used to generate project/make files - in your example it has produced `Makefile` which are used to build your software (mostly on Linux/Unix platform).

Cmake allows to provide cross platform build files that would generate platform specific project/make files for particular compilation/platform.

For instance you may to try to compile your software on Windows with Visual Studio then with proper syntax in your CMakeLists.txt file you can launch `cmake .` inside your project's directory on Windows platform,Cmake will generate all the necessary project/solution files (`.sln` etc.).

If you would like to build your software on Linux/Unix platform you would simply go to source directory where you have your `CMakeLists.txt` file and trigger the same `cmake .` and it will generate all files necessary for you to build software via simple `make`.

If you'd like to make platform dependent library includes / variable definitions etc. you can use this syntax in CMakeLists.txt file:

```
IF(WIN32)
   ...do something...
 ELSE(WIN32)
   ...do something else...
 ENDIF(WIN32)
```

- CMake sets your project up according to the root `CMakeLists.txt` of your project, and does so in whatever directory you executed `cmake` from in the console. It will create a bunch of files, but the only thing you have to worry about is the Makefile and the project folders. If you have, for example, one folder for the source code and another for the library you also use, you would want to have two `CMakeLists.txt` files for every folder: they will tell the `make` command how to build the actual executable. Then, the project is built, and the executable is ready to be executed. 

### Including (and compiling) custom libraries

- Once we create our custom library, called `glib` which is defined using `glib/glib.h` (interface) and `glib/glib.cpp` (functions implementation), we need to tell CMake how to create the executables for main and glib. The directory is the following:

```
project/
├─ glib/
|   ├─ glib.h
|   ├─ glib.cpp
├─ main.cpp
```

where `main.cpp` uses functions in `glib.cpp`. How do we compile both and tell CMake that one depends on another? First, there must be two `CMakeLists.txt`: one to compile the main and one for glib. The three should be:

```
project/
├─ glib/
|   ├─ glib.h
|   ├─ glib.cpp
|   ├─ CMakeLists.txt
├─ main.cpp
├─ CMakeLists.txt
```

The `CMakeLists.txt` should look like this:

```
cmake_minimum_required(VERSION 2.8)
project( main )
find_package( OpenCV REQUIRED )
include_directories( ${OpenCV_INCLUDE_DIRS} )
add_subdirectory(glib)
add_executable( main main.cpp )
target_link_libraries( main ${OpenCV_LIBS} glib)
```

where `add_subdirectory(glib)` includes a subdir to the build. The parameter specifies the directory in which the source `CMakeLists.txt` and code files are located. The command `target_link_libraries()` should contain a parameter which must have been created by a command `add_library()`, which is in the `CMakeLists.txt` file inside the `glib/` folder. This file looks like below:

```
add_library(glib OBJECT
    glib.cpp
    glib.h
)
target_include_directories(glib
    PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/../glib
)
```

The command `add_library(<target>)` adds a library called `<target>` to the build from the source files listed in the command. The name `<target>` is the logical target name and must be unique in a project. The command `target_include_directories(<target>)` specifies include directories to use when compiling a given target, and the name `<target>` must have been created by a command such as `add_executable()` or `add_library()`. 