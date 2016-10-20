#Nao with cmake files

This project and `helloworld_static` have been created to do the same 
purpose than the tutorial written in `README.md` which you can find in `/nao_robot`.

The difference is `nao_turorial` use directly the compiler gcc with 
the necessary flags to compile, and `helloworld_static` and `helloworld` 
are doing the same but using CMake.

`helloworld_static` is for having all the libraries static and `helloworld`
has only static libgcc libc++ and rapp.

In this example we'll run an RAPP API project in NAO robot. *Only using RAPP libraries*

##Code

In this example we are going to use the code of `available_services` example, 
because it's the easiest one. 

```cpp
#include <rapp/cloud/service_controller.hpp>
#include <rapp/cloud/available_services.hpp>
#include <iostream>
/*
 * \brief Example to show how available_services works
 */
int main(int argc, char* argv[])
{

	rapp::cloud::platform info = {"155.207.19.229", "9001", "rapp_token"}; 
	rapp::cloud::service_controller ctrl(info);

	auto cb = [](std::vector<std::pair<std::string, std::string>> services) {
         std::cout << "available services: " << std::endl;
         for (const auto & pair : services) {
            std::cout << pair.first << " " << pair.second << std::endl;
         }
     };

    ctrl.make_call<rapp::cloud::available_services>(cb);
	return 0;
}
```

This is going to show all the services that you can use with RAPP API.

##CMakeLists

Like we said before, we are going to use only RAPP libraries, so this file will be similar to
`available_services` example.

```
cmake_minimum_required(VERSION 2.6)

project(helloworld)

add_executable(helloworld source/helloworld.cpp)

set(LIBRARY_PATH ${LIBRARY_PATH} /usr/local/lib)

find_package(OpenSSL REQUIRED)
find_package(Boost COMPONENTS system REQUIRED)
find_package(Threads REQUIRED)

set(CMAKE_FIND_LIBRARY_SUFFIXES ".a")
find_library(RAPP_LIBRARY NAMES rapp REQUIRED)
message(STATUS "${RAPP_LIBRARY}")

set(RAPP_LIBRARIES ${RAPP_LIBRARY} 
                   ${OPENSSL_LIBRARIES} 
				   ${Boost_LIBRARIES}
				   ${CMAKE_THREAD_LIBS_INIT})

target_link_libraries(helloworld ${RAPP_LIBRARIES})

set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS}-static-libstdc++ -static-libgcc")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++1y -Wall -fPIC -Os -s -O2 -march=atom -mtune=atom -mfpmath=sse")
```

You could see that we added one line before trying to find RAPP library:

```
set(CMAKE_FIND_LIBRARY_SUFFIXES ".a")
```

With that line you are looking for the static RAPP library. 
This is to avoid install RAPP library in NAO. 
The rest of the examples about NAO robot are going to use always the static one.

And the last difference is the `CMAKE_CXX_FLAGS`. Besides the type of c++ that we are going to use
we added some optimization flags especially for NAO which is based on Intel Atom processor.
*Availables since gcc 4.5*

If you need more information about the CMakeLists.txt you can see the `helloworld` tutorial in the begginers tutorial folder.

##Build and Run

Now we have to save and close the files, and we go to the terminal.
In there we'll do the next steps:

```
mkdir build
cd build
cmake ..
make
```

After all, you'll have your executable in `build/` folder.

Finally, we have to move/copy our executable to NAO robot. 
The easy way to do this is using ssh when you are connected to your robot:

```
scp path/to/your/executable nao@ip_of_your_NAO:/path/to/folder/you/need
```

You'll have to change the values of the paths and IP of the robot.

Now you can run your example in NAO and see all the services that RAPP offers.
