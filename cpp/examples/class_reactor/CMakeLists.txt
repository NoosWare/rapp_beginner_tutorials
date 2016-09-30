cmake_minimum_required(VERSION 2.6)

project(class_reactor)

add_executable(class_reactor source/class_reactor.cpp)

set(LIBRARY_PATH ${LIBRARY_PATH} /usr/local/lib)

find_library(RAPP_LIBRARY NAMES rapp REQUIRED)
find_package(OpenSSL REQUIRED)
find_package(Boost 1.55 COMPONENTS system REQUIRED)
find_package(Threads REQUIRED)

set(RAPP_LIBRARIES ${RAPP_LIBRARY} 
                   ${OPENSSL_LIBRARIES} 
				   ${Boost_LIBRARIES}
				   ${CMAKE_THREAD_LIBS_INIT})

target_link_libraries(class_reactor ${RAPP_LIBRARIES})

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
