cmake_minimum_required(VERSION 2.8)
project(helloworld)

find_package(qibuild)

set(CMAKE_CXX_FLAGS 
"-std=gnu++1y -static-libstdc++ -march=atom -mtune=atom -mfpmath=sse")

find_package(rapp)
message(STATUS "libraries: ${RAPP_LIBRARIES}")
message(STATUS "headers: ${RAPP_INCLUDE_DIRS}")

qi_create_bin(helloworld "main.cpp")
qi_use_lib(helloworld ALCOMMON ALPROXIES openssl RAPP boost_system boost_thread pthread)
