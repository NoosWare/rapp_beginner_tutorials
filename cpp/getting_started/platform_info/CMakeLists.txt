cmake_minimum_required(VERSION 2.6)

project(config)

add_executable(config source/config.cpp)

set(LIBRARY_PATH ${LIBRARY_PATH} /usr/local/lib)

find_library(RAPP_LIBRARY NAMES rapp REQUIRED)
find_package(OpenSSL REQUIRED)
find_package(Boost COMPONENTS system REQUIRED)
find_package(Threads REQUIRED)

set(RAPP_LIBRARIES ${RAPP_LIBRARY} 
                   ${OPENSSL_LIBRARIES} 
				   ${Boost_LIBRARIES}
				   ${CMAKE_THREAD_LIBS_INIT})

target_link_libraries(config ${RAPP_LIBRARIES})

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
