---
layout: post
title:  "Use More Than One Library With CGO"
date:   2024-06-27 09:10:00 -0600
categories: cuda
modified_date:   2024-06-27 10:20:00 +0000
--- 

Using a shared library written in C (.so file for Linux) wasn't trivial and because of that, I wrote a post here. Now, there's a need for another library written in C to be used with CGo. And once again, I witnessed some behaviours that made me lose my mind. Therefore, here's another post on how to use more than one C library in Go.

The project had been using one shared lib in C so far. It had the following structure:

```
project/
├─ cuda/
|  ├─ wrapper/
|  |  ├─ WrapperGPU.c
|  |  ├─ WrapperGPU.h
|  |  ├─ CMakeLists.txt
|  ├─ main.cu
|  ├─ MainInterface.hpp
|  ├─ CMakeLists.txt
|  ├─ ...
```

Where key files for bridging with go where:

- **WrapperGPU.h**

```c++
#ifndef WRAPPERGPU_H
#define WRAPPERGPU_H

#include <stdlib.h>
#include "Types.h"

void WrapGenerateBatch(const char*, int*, int*, MassGenItem*, char*, int, int, uint8_t*);

#endif
```

- **WrapperGPU.c**

```c++
#include <stdio.h>
#include "../MainInterface.hpp"
#include "WrapperGPU.h" 

void WrapGenerateBatch(
	const char* storageDir, 
	int* gridConfig, 
	int* blockConfig, 
	MassGenItem* objects, 
	char* callToAction, 
	int callToActionSize, 
	int startIndex,
	uint8_t* errorsPtr
) {
	generate(
		storageDir,
		gridConfig, 
		blockConfig, 
		objects, 
		callToAction, 
		callToActionSize, 
		startIndex,
		errorsPtr
	);
}
```

- **MainInterface.hpp**

```c++
#ifndef MAIN_INTERFACE_H
#define MAIN_INTERFACE_H

#include "wrapper/Types.h"

#ifdef __cplusplus
extern "C" {
#endif

void generate(const char*, int*, int*, MassGenItem*, char*, int, int, uint8_t*);

#ifdef __cplusplus
}
#endif

#endif
```

- **main.cu**

```c++
#include "MainInterface.hpp"
extern "C" {
    void generate(...) {

    }
}
```

Now we want use another library in C. We must copy or have the **folder structure** from the C project in our current project, and locate the .so file. Imagine the structure now is:

```
project/
├─ cuda/
|  ├─ wrapper/
|  |  ├─ WrapperGPU.c
|  |  ├─ WrapperGPU.h
|  |  ├─ CMakeLists.txt
|  ├─ main.cu
|  ├─ MainInterface.hpp
|  ├─ CMakeLists.txt
├─ wrapper/
|  ├─ third_party/
|  |  ├─ lib2/
|  |  |  ├─ c_project/
|  |  |  |  ├─ wrapper/
|  |  |  |  |  ├─ WrapperCPU.c
|  |  |  |  |  ├─ WrapperCPU.h
|  |  |  |  |  ├─ CMakeLists.txt
|  |  |  |  ├─ main.cpp
|  |  |  |  ├─ CMakeLists.txt
|  ├─ main.go
```

Imagine the bridge files in this new lib to be like this:

- **WrapperCPU.h**

```c++
#ifndef WRAPPER_H
#define WRAPPER_H

#include "Types.h"

int WrapGenerateSingle(RequestInfo*, unsigned char**, size_t*, const char*, const char*, const char*, const char*, const char*);

#endif
```

- **WrapperCPU.c**

```c++
#include "../MainInterface.hpp"
#include "WrapperCPU.h" 
#include <stdio.h>
#include <stddef.h>

int WrapGenerateSingle(
	RequestInfo* req, 
	unsigned char** dataPtr, 
	size_t* dataSize,
	const char* assetsFolderPath,
	const char* normalFontPath,
	const char* boldFontPath,
	const char* companyLogoPath,
	const char* extraLogoPath
) {
	int error = generate(
		req, 
		dataPtr, 
		dataSize,
		assetsFolderPath,
		normalFontPath,
		boldFontPath,
		companyLogoPath,
		extraLogoPath
	);
	return error;
}
```

- **MainInterface.hpp**

```c++
#ifndef MAIN_INTERFACE_H
#define MAIN_INTERFACE_H

#include "wrapper/Types.h"
#include <stddef.h>

#ifdef __cplusplus
extern "C" {
#endif

int generate(RequestInfo*, unsigned char**, size_t*, const char*, const char*, const char*, const char*, const char*);

#ifdef __cplusplus
}
#endif

#endif
```

- **main.cu**

```c++
#include "MainInterface.hpp"
extern "C" {
    void generate(...) {

    }
}
```

We include this `libwrapperCPU.so` in another .go file separate from the one that includes the `libwrapperGPU.so`, but notice that both libraries have a definition for a function named `generate()`. This functions must be in the symbol table so that golang recognizes it within CGo. By having both the same name `generate()`, CGo **will mistakenly call both at the same time** when you call either `WrapGenerateCPU` or `WrapGenerateGPU`.

This error had me scratching my head for hours. In order to fix it, we must change `generate()` function name in the bridging files.

### Solution: Bridge Files for Lib1

- **WrapperGPU.h**

```c++
#ifndef WRAPPERGPU_H
#define WRAPPERGPU_H

#include <stdlib.h>
#include "Types.h"

void WrapGenerateBatch(const char*, int*, int*, MassGenItem*, char*, int, int, uint8_t*);

#endif
```

- **WrapperGPU.c**

```c++
#include <stdio.h>
#include "../MainInterfaceGPU.hpp" // <--- include the new header
#include "WrapperGPU.h" 

void WrapGenerateBatch(
	const char* storageDir, 
	int* gridConfig, 
	int* blockConfig, 
	MassGenItem* objects, 
	char* callToAction, 
	int callToActionSize, 
	int startIndex,
	uint8_t* errorsPtr
) {
	generateGPU( // <--- call new function
		storageDir,
		gridConfig, 
		blockConfig, 
		objects, 
		callToAction, 
		callToActionSize, 
		startIndex,
		errorsPtr
	);
}
```

- **MainInterfaceGPU.hpp** (change the file name)

```c++
#ifndef MAIN_INTERFACE_GPU_H // <--- unique directive name
#define MAIN_INTERFACE_GPU_H // <--- unique directive name

#include "wrapper/Types.h"

#ifdef __cplusplus
extern "C" {
#endif

void generateGPU(const char*, int*, int*, MassGenItem*, char*, int, int, uint8_t*); // <--- new signature

#ifdef __cplusplus
}
#endif

#endif
```

- **main.cu**

```c++
#include "MainInterfaceGPU.hpp" // <--- include new file
extern "C" {
    void generateGPU(...) { // <--- implement new signature

    }
}
```

### Solution: Bridge Files for Lib2

- **WrapperCPU.h**

```c++
#ifndef WRAPPERGPU_H
#define WRAPPERGPU_H

#include <stdlib.h>
#include "Types.h"

void WrapGenerateBatch(const char*, int*, int*, MassGenItem*, char*, int, int, uint8_t*);

#endif
```

- **WrapperCPU.c**

```c++
#include "../MainInterfaceCPU.hpp" // <--- include new header
#include "WrapperCPU.h" 
#include <stdio.h>
#include <stddef.h>

int WrapGenerateSingle(
	RequestInfo* req, 
	unsigned char** dataPtr, 
	size_t* dataSize,
	const char* assetsFolderPath,
	const char* normalFontPath,
	const char* boldFontPath,
	const char* companyLogoPath,
	const char* extraLogoPath
) {
	int error = generateCPU( // <--- call new function
		req, 
		dataPtr, 
		dataSize,
		assetsFolderPath,
		normalFontPath,
		boldFontPath,
		companyLogoPath,
		extraLogoPath
	);
	return error;
}
```

- **MainInterfaceCPU.hpp** (change the file name)

```c++
#ifndef MAIN_INTERFACE_CPU_H // <--- unique directive
#define MAIN_INTERFACE_CPU_H // <--- unique directive

#include "wrapper/Types.h"
#include <stddef.h>

#ifdef __cplusplus
extern "C" {
#endif

int generateCPU(RequestInfo*, unsigned char**, size_t*, const char*, const char*, const char*, const char*, const char*); // <--- new signature

#ifdef __cplusplus
}
#endif

#endif
```

- **main.cu**

```c++
#include "MainInterfaceCPU.hpp" // <--- include new header
extern "C" {
    void generateCPU(...) { // <--- implement new signature

    }
}
```

Change the CMakeLists.txt files accordingly so that the `MainInterface.hpp` file name changes don't affect the compilation, and the problem is solved.

Basically having both functions named `generate()` (and both `MainInterface.hpp` files called the same) confused the dynamic linking since maybe somewhere in the symbol table CGo saw already one of the `generate()` functions and thought it was the one we where calling.

Now we can include these libs in two different files like this:

- **singles/singles.go**

```go
// +build single

// #cgo CFLAGS: -I../third_party/lib2/c_project/wrapper
// #cgo LDFLAGS: -L../third_party/lib2/c_project/wrapper -lwrapperCPU
// #include <stdlib.h>
// #include <stddef.h>
// #include "WrapperCPU.h"
import "C"

func singleCPU(){
    status := C.WrapGenerateSingle(...)
}
```

- **batch/batch.go**

```go
// +build batch

// #cgo CFLAGS: -I../../cuda/wrapper
// #cgo LDFLAGS: -L../../cuda/wrapper -lwrapperGPU
// #include <string.h>
// #include "WrapperGPU.h"
import "C"
```

Notice that in CMakeLists.txt of each project's wrapper/ folder, you must modify a few things so that the libs are also **not called the same**, such as `libwrapperCPU.so` and `libwrapperGPU.so`. In the other post is detailed on how to set the name of your .so file.