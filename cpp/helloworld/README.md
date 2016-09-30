#Getting Started

**This tutorial assumes that RAPP API is installed and built**

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
The next step is to set the project name. In this case, we named `helloworld`:

```
project(helloworld)
```

Now, we need to have an executable of our code to check that everything is working
in the way that we want. So we have to add:

```
add_executable(helloworld source/helloworld.cpp)
```

In this line you have to say first how you want to name your executable, in our
case we called it `helloworld` too. And after that, you have to specify the relative 
path of the file that you want to make executable.

Until this part, we have the basic file done.
Now we have to associate the libraries.

##Associate with the libraries

In this part we are going to associate our files with **statics** and **shared** libraries.

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

Then, the next part that we have to add to our document is this:

```
find_library(RAPP_LIBRARY NAMES rapp REQUIRED)
find_package(OpenSSL REQUIRED)
find_package(Boost 1.55 COMPONENTS system REQUIRED)
find_package(Threads REQUIRED)
```

You can see that there is a difference between the first line and the rest: we don't use the 
same command. This is RAPP library you can't find it like a package, so we have to use the command
`find_library`. With this command, first you call your library with the name that you want, in this case
`RAPP_LIBRARY`, and cmake has to look for libraries with the name `rapp`. The word `REQUIRED` imply that
is the library is not found, there will be an error and we won't be able to execute our program.

The command `find_packages` is similar to the previous one. The only detail is that you only have to 
indicate the name of the package that the library has. You can appreciate when we are looking for 
`Boost` package, it is different. This is because some libraries come with one or more dependent libraries 
or components. In this case we have the specific version of the library (1.55) and it needs `system` component.

*For more information you can visit the following links:*

[Find-library-command](https://cmake.org/cmake/help/v3.0/command/find_library.html?highlight=find_library)

[Find-package-command](https://cmake.org/cmake/help/v3.0/command/find_package.html?highlight=find_package)

The last step for finishing with the libraries is to link our project to them. For to do that, we have two ways 
to do it:

1. Put all the libraries in target_link_libraries command

```
target_link_libraries(helloworld ${RAPP_LIBRARY} 
                      ${OPENSSL_LIBRARIES} 
                      ${Boost_LIBRARIES}
                      ${CMAKE_THREAD_LIBS_INIT})
```

In this lines you can see that we are saying that our **executable** is linked to this libraries, which
we are looking for in the previous step. *NOTE:* The name of the libraries which are not RAPP, they are 
define for the package. In the case you need other package library you have to look for its exact name
in cmake.

2. Define all the libraries first

```
set(RAPP_LIBRARIES ${RAPP_LIBRARY} 
                   ${OPENSSL_LIBRARIES} 
                   ${Boost_LIBRARIES}
                   ${CMAKE_THREAD_LIBS_INIT})

target_link_libraries(helloworld ${RAPP_LIBRARIES})
```

We define first all the libraries in the parameter `RAPP_LIBRARIES`, so we only have to link this one
with the `target_link_libraries` command.
We use the last one, you can choose the way that you prefer.

*For more information you can visit the following link:*

[Target-link-libraries command](https://cmake.org/cmake/help/v3.0/command/target_link_libraries.html)

##CMake Flags

If you need to specify some information to the compiler, you have to do that with cmake flags.
In our case, we are going to compile in c++, so the command needed is `CMAKE_CXX_FLAGS`.
The only detail that we have to specify is that we need to compile our code in C++14, so
we have to add the next line to our code:

```
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
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

1. Go to your project path (in our case `helloworld/`)
2. Build your project
```
mkdir build
cd build 
cmake ..
```

3. If everything is ok, you will have created your executable `helloworld` in the folder build.
4. Run your executable
    ```
    ./helloworld
    ```

5. If there aren't problems, you will have created a `helloworld.wav` in the same folder.

Now you can explore and make your own projects!
