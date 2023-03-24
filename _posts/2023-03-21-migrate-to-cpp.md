---
layout: post
title:  "Migrate OpenCV System To C++ (Server)"
date:   2023-03-21 09:30:00 -0600
categories: cmake
modified_date:   2023-03-21 11:30:00 +0000
---

## Build a C++ OpenCV Server System with CMake

Python was the language with which the embedded QR code system was prototyped, but it proved to be quite slow. Therefore, the system was migrated entirely to C++ and it worked like magic: it finished executing almost immediately. Some benchmarks on the comparison between the Python and C++ system will be posted soon.

So, since now the system runs on C++, how do we manage all the custom libraries? With **CMake**, since it takes care of the build pipeline for all the dependency executables and links. Here are some of my notes regarding CMake with OpenCV and custom libraries.

Cmake is cross-platform, open-source build system for managing the build process of software using a compiler-independent method. In most cases it is used to generate project/make files, hence it has produced `Makefile` which are used to build your software (mostly on Linux/Unix platform).

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

where `main.cpp` uses functions in `glib.cpp`. How do we compile both and tell CMake that one depends on another? First, there must be two `CMakeLists.txt`: one to compile the main and one for glib. The tree should be:

```
project/
├─ glib/
|   ├─ glib.h
|   ├─ glib.cpp
|   ├─ CMakeLists.txt
├─ main.cpp
├─ CMakeLists.txt
```

The `./CMakeLists.txt` should look like this:

```
cmake_minimum_required(VERSION 2.8)
project( main )
find_package( OpenCV REQUIRED )
include_directories( ${OpenCV_INCLUDE_DIRS} )
add_subdirectory(glib)
add_executable( main main.cpp )
target_link_libraries( main ${OpenCV_LIBS} glib)
```

where `add_subdirectory(glib)` includes a subdir to the build. The parameter specifies the directory in which the source `CMakeLists.txt` and code files are located. The command `target_link_libraries()` should contain a parameter which must have been created by a command `add_library()`, which is in the `CMakeLists.txt` file inside the `glib/` folder. The file `./glib/CMakeLists.txt` looks like below:

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

So far, the folder tree looks as follows:

```
c++/
├─ glib/
|   ├─ glib.h
|   ├─ glib.cpp
|   ├─ CMakeLists.txt
├─ colorsys/
|   ├─ colorsys.h
|   ├─ colorsys.cpp
|   ├─ CMakeLists.txt
├─ light/
|   ├─ light.h
|   ├─ light.cpp
|   ├─ CMakeLists.txt
├─ noise/
|   ├─ noise.h
|   ├─ noise.cpp
|   ├─ CMakeLists.txt
├─ utils/
|   ├─ utils.h
|   ├─ utils.cpp
|   ├─ CMakeLists.txt
├─ writer/
|   ├─ writer.h
|   ├─ writer.cpp
|   ├─ CMakeLists.txt
├─ main.cpp
├─ CMakeLists.txt
```

The folder `c++/writer/` contains the algorithm, which is used by main. Therefore, the `writer.cpp` must include the header files for the libraries: `glib.h`, `colorsys.h`, `light.h`, `noise.h`, since it contains the main image processing algorithm. Then, main includes the header `writer.h` to get access to said algorithm. 

To include a bunch of libraries in one file (writer) and then use it as another dependency inside main, the CMake files were a bit modified:

- `c++/CMakeLists.txt`:

```
cmake_minimum_required(VERSION 2.8)
project( main )
find_package( OpenCV REQUIRED )
include_directories( ${OpenCV_INCLUDE_DIRS} )
add_subdirectory(glib)
add_subdirectory(utils)
add_subdirectory(noise)
add_subdirectory(light)
add_subdirectory(colorsys)
add_subdirectory(writer)
add_executable( main main.cpp )
target_link_libraries( main ${OpenCV_LIBS} glib utils noise light colorsys writer)
```

- `c++/writer/CMakeLists.txt`:

```
add_library(writer OBJECT
    writer.cpp
    writer.h
)
target_include_directories(writer
    PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/../writer
)

target_include_directories(writer PUBLIC "../glib/" "../utils/" "../light/" "../colorsys/")
```

- All the end libs CMakes (noise, light, utils, etc) that are only used by `writer.cpp`, maintain the same structure:

```
add_library(noise OBJECT
    noise.cpp
    noise.h
)
target_include_directories(noise
    PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/../noise
)
```

Then, everything is compiled and executed as:

```
cmake .
make
./main
```

## C++ Server

This system will create a QR and send it through an API, and therefore the API should be in a server. C++ is used for low-latency server, and we will find out what does that mean.

**Sockets** let apps attach to the local networks at different **ports**.

> A web Socket is a computer communications protocol, providing full-duplex communication channels over a single TCP connection. 

### IP Address

As previously mentioned, the IP address is a unique address within a computer network that allows sending data to the intended receiver. They are not bound to a specific physical location but rather to a specific device. The IP defines the package structure by adding also the IP Header to the package.

It’s a 32-bit number that uniquely identifies a host within a TCP/IP computer network. A host can be any device connected to the network such as a computer, a phone or a printer. The IP address is described in the dotted-decimal format: 192.168.123.32.

The IP address has two parts:

1. The network part representing the destination network of the data sent.
2. The host part representing the destination host in the destination network.

### Subnet Masks

A router can use an additional 32-bit number to infer whether the destination host is part of the local subnet. This number is the **subnet mask**. It may look something like: 255.255.255.0. If we look at the binary version of the IP address and the subnet mask,

> 11000000.10101000.01111011.10000100 (192.168.123.32)

> 11111111.11111111.11111111.00000000 (255.255.255.0)

The positions where the subnet mask is set to 1 tell us where the network part of the IP address is. Thus, they also tell us where the host part is, with all the positions where the subnet mask is 0.

In a network, a package is sent to its destination thanks to routers.The router infers the network that the host is part of by considering the network part of the IP address. It uses its routing table to compute the best way to forward the package to its destination network. Once the destination network is reached, the local router sends the package to the host by reading the host part of the IP address.

Routers and switches rely on sbunet masks to route data packets to its destinations. A package presents its IP address, and for any router in the network, the IP address will be processed with binary operations using the subnet mask. These operations allow the router to see the IP host address of the packet's destination, and with it the router knows to which subnet it should send the package.

### C++ Server

- **Sockets** are the endpoints that receive data within a network.Server sockets communicate with client sockets.

```c++
// create the socket
int listening = socket(AF_INET, SOCK_STREAM, 0);
if (listening == -1)
{
  std::cerr << "Can't create a socket!";
  return -1;
}
```

`AF_INET` and `SOCK_STREAM` are constants that represent the IPv4 address family and a TPC connection, respectively. Then, we need to **bind** the socket, which is telling the socket to wich address and port it should connect. For that, we need a socket address object called **hint**, and set the IP address family, the port and the IP address:

```c++
// binding
struct sockaddr_in hint;
hint.sin_family = AF_INET;
hint.sin_port = htons(54000);
inet_pton(AF_INET, "0.0.0.0", &hint.sin_addr);
```

The "0.0.0.0" IP address is not a specification, but it lets the server decide on which address it's going to listen to client connections. The function `htons()` transalte the interger port into a network byte order number.

```c++
// binding
if (bind(listening, (struct sockaddr *)&hint, sizeof(hint)) == -1) 
{
    std::cerr << "Can't bind to IP/port";
    return -2;
}
```

By having the socket bind to an IP address and port, we can now tell it to listen to incoming connections:

```c++
// listen
if (listen(listening, SOMAXCONN) == -1)
{
  std::cerr << "Can't listen !";
  return -3;
}
```

The constant `SOMAXCONN` defines the maximum number of incoming connections: it is a system-defined value that represents the maximum number of connections allowed to be queued for the socket. In most modern operating systems, `SOMAXCONN` is typically set to a high value, such as 128 or 512, which means that the server can handle that many simultaneous connections in the queue. 

This server, however, needs to listen to connections forever, not just once. So we need to modify the main function:

```c++
int s = serve(image, qr, qrColor);
if (s < 0){
  printf("[ERROR] Server could not serve.\n");
} else {
  printf("[SUCCESS] Server listening.\n");
}
```

and add the `serve()` function:

```c++
int serve(Mat image, Mat qr, Mat qrColor){
    // create the socket code

    // binding the socket codes

    // socket listening code

    char buf[4096];
    while (true) {
        sockaddr_in client;
        socklen_t clientSize = sizeof(client);

        int clientSocket = accept(listening, (sockaddr*)&client, &clientSize);

        if (clientSocket == -1) {
            cerr << "Problem with client connection\n";
            continue;
        }

        memset(buf, 0, 4096);
        int bytesReceived = recv(clientSocket, buf, 4096, 0);
        if (bytesReceived == -1) {
            cerr << "Error in recv(). Quitting\n";
            break;
        }

        if (bytesReceived == 0) {
            cerr << "Client disconnected\n";
            break;
        }

        cout << string(buf, 0, bytesReceived) << endl;

        // return an image (output of the algorithm)
        // Convert image to PNG format
        Mat out = Writer::generateQR(image, qr, qrColor);
        vector<uchar> buffer;
        imencode(".png", out, buffer);
        string imageString(buffer.begin(), buffer.end());

        // Construct HTTP response
        ostringstream oss;
        oss << "HTTP/1.1 200 OK\r\n";
        oss << "Content-Type: image/png\r\n";
        oss << "Content-Length: " << imageString.size() << "\r\n";
        oss << "\r\n";
        oss << imageString;

        string httpResponse = oss.str();

        // Send HTTP response
        send(clientSocket, httpResponse.c_str(), httpResponse.size() + 1, 0);

        close(clientSocket);
    }

    close(listening);
    return 0;
}
```

The code snippet:

```c++
// Construct HTTP response
std::ostringstream oss;
oss << "HTTP/1.1 200 OK\r\n";
oss << "Content-Type: image/png\r\n";
oss << "Content-Length: " << imageString.size() << "\r\n";
oss << "\r\n";
oss << imageString;
```

creates the response string that will be sent back, which includes some headers with the extension and length of the image string. `imageString` object was done using `imencode` and `imageString` functions.

- The function `imencode` compresses the image and stores it in the memory buffer that is resized to fit the result. It converts (encodes) image formats into streaming data and stores it in-memory cache or RAM instead of disk. It is mostly used to compress image data formats in order to make network transfer easier. It creates an array of bytes ordered in a sequence defined by the format. Each format (PNG, JPG, etc) has its own serialization conventions. It also contains some meta-data related to that image format, ex: compression level, etc. along with pixel data.

Now the server is ready to locally return the output image to a client connected to the same network and that sends a GET request to its socket.

![img]({{site.url}}/img/6/sc01.png)