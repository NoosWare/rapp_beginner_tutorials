#Getting Started

**This tutorial assumes that RAPP API is installed and built with -DRAPP_STATIC=ON flag**

In the following document it is going to explain how to create your own project based in RAPP
and how to do a proper CMakeLists.txt

Using CMake requires you to configure your CMakeLists.txt in the **root** of your RApp project.
Essentially, the structure is the following:

```
project/
        CMakeLists.txt
        sources/
                helloworld.cpp
        build/
```

## LIMITATIONS

**WARNING** 
Because we do a static link to GLIBC, the underlying `boost_system` library cannot use `getaddrinfo`,
therefore there is no domain name resolution to IPs.
Hence, full static linking **requires** you to use an IP only!


##Writing your code

The first step is to write your code. 
In this case, we created a project called `helloworld` with a folder `source` where we have our example call `helloworld.cpp`.
You can see [here](source/helloworld.cpp).

In this example we use RAPP for writing a text and load that text in a wav file.

##Writing CMakeLists.txt

When our code is written, we can do the CMakeLists file.
We have to write it in the directory `helloworld`.

First of all is the version of our cmake, it is said in the following line:

```
cmake_minimum_required(VERSION 2.6)
```

In our case, the minimum version is 2.6. You can change depending of your requirements.
The next step is to set the project name. In this case, we named `helloworld_static`:

```
project(helloworld_static)
```

Now, we need to have an executable of our code to check that everything is working
in the way that we want. So we have to add:

```
add_executable(helloworld_s source/helloworld.cpp)
```

In this line you have to say first how you want to name your executable, in our
case we called it `helloworld_s`. And after that, you have to specify the relative 
path of the file that you want to make executable.

Until this part, we have the basic file done.
Now we have to associate the libraries.

##Linking with libraries

In this part we are going to associate our files with **statics** libraries.

The first step is to indicate the directory where the libraries are installed.
If RAPP API is correctly installed, it will be in `/usr/local/lib`, so 
we have to put the next line:

```
set(LIBRARY_PATH ${LIBRARY_PATH} /usr/local/lib)
```

In this line you are setting the parameter `LIBRARY_PATH` as `usr/local/lib`.
This `LIBRARY_PATH` technique will affect all `find_library` commands after you set the variable.

The next step is to look for specific libraries, in our case we have four:

* RAPP
* OpenSSL
* Boost
* Threads

To make sure that we are looking for static libraries instead of the shared, we are going to
add this line:

```
set(CMAKE_FIND_LIBRARY_SUFFIXES ".a")
```

*NOTE:* The shared libraries finish in `.so`, not in `.a`.

The problem of make static a library is that you have to do static all the dependecies of 
the libraries you are using. So, in the first place we are going to look for the basic 
libraries:

```
set(LIBDL_NAMES dl libdl ltdl libltdl)
find_library(LIBDL_LIBRARY NAMES ${LIBDL_NAMES})
message(STATUS ${LIBDL_LIBRARY})
```

The last line only shows if the libraries have been found.
In the same way, we look for RAPP and OpenSSL:

```
find_library(RAPP_LIBRARY NAMES rapp REQUIRED)
message(STATUS "${RAPP_LIBRARY}")

find_package(OpenSSL REQUIRED)
message(STATUS "${OPENSSL_LIBRARIES}")

find_package(Threads REQUIRED)
```

You can see that there is a difference between looking for rapp library and the rest: we don't use the 
same command. This is because in RAPP library you can't find it like a package, so we have to use the command
`find_library`. With this command, first you call your library with the name that you want, in this case
`RAPP_LIBRARY`, and cmake has to look for libraries with the name `rapp`. The word `REQUIRED` imply that
is the library is not found, there will be an error and we won't be able to execute our program.

The command `find_packages` is similar to the previous one. The only detail is that you only have to 
indicate the name of the package that the library has. You can see with `Boost` library.

*For more information you can visit the following links:*

[Find-library-command](https://cmake.org/cmake/help/v3.0/command/find_library.html?highlight=find_library)

[Find-package-command](https://cmake.org/cmake/help/v3.0/command/find_package.html?highlight=find_package)

Then we continue looking for our libraries:

```
find_package(Boost 1.55 COMPONENTS system REQUIRED)
set(Boost_USE_STATIC_LIBS ON)
set(BUILD_SHARED_LIBS OFF)
include_directories(${Boost_INCLUDE_DIR})
message(STATUS "${Boost_SYSTEM_LIBRARY}")
```

In this case, `Boost` has its own way to do it static.  Besides, you can appreciate when we are looking for 
`Boost` package, it is different of the previous one. This is because some libraries come with one or more dependent libraries 
or components. In this case we have the specific version of the library (1.55) and it needs `system` component.


The last step for finishing with the libraries is to link our project to them:

```
target_link_libraries(helloworld_s ${RAPP_LIBRARY} 
                                   ${OPENSSL_LIBRARIES} 
								   ${Boost_LIBRARIES}
								   ${CMAKE_THREAD_LIBS_INIT}
								   ${LIBDL_LIBRARY})
```

In this lines you can see that we are saying that our **executable** is linked to this libraries, which
we are looking for in the previous steps. *NOTE:* The name of the libraries which are not RAPP, they are 
define for the package. In the case you need other package library you have to look for its exact name
in cmake.

*For more information you can visit the following link:*

[Target-link-libraries command](https://cmake.org/cmake/help/v3.0/command/target_link_libraries.html)

##CMake Flags

If you need to specify some information to the compiler, you have to do that with cmake flags.
In our case, we are going to compile in c++, so the command needed is `CMAKE_CXX_FLAGS`.
The only detail that we have to specify is that we need to compile our code in the newest versions
of C++ (C++11 and C++14), so we have to add the next line to our code:

```
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++1y")
```

We can't specify for C++14 because there are dependencies of the libraries which need C++11.
Besides, it is necessary to put a flag which says that all the links that we have created are
static:

```
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS}-static")
```

Finally, our CMakeLists.txt is done. 

##Repository detail

Before to do anything we have to be careful in the case that we are using a repository.
If you are not using with your project, then ignore this part.
In the case you are using one with Github, you will have to create a file call `.gitignore`
(inside your project directory) where you only write this:

```
build/
```

This means that you are not going to save this folder in the repository. It is good to do it
because in the case you shared your project, the other person won't have problems building it.
It's a specific folder for every user.

##Building your code

The next step is about create our `build` folder and run our code.

Now we are going to work in the terminal.

1. Go to your project path (in our case `helloworld_static/`)
2. Build your project
```
mkdir build
cd build 
cmake ..
```

3. If everything is ok, you will have created your executable `helloworld_s` in the folder build.
4. Run your executable
```
./helloworld_s
```

5. If there aren't problems, you will have created a `helloworld.wav` in the same folder.

Now you can explore and make your own projects!
