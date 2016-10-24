cmake_minimum_required(VERSION 2.6)

project(qr_recognition)

add_executable(qr_recognition source/qr_recognition)

set(LIBRARY_PATH ${LIBRARY_PATH} /usr/local/lib)

find_library(RAPP_LIBRARY NAMES rapp REQUIRED)
find_package(OpenSSL REQUIRED)
if(OPENSSL_FOUND)
    include_directories(${OPENSSL_INCLUDE_DIR})
    message(STATUS "Using OpenSSL Version: ${OPENSSL_VERSION}")
	message(STATUS "OpenSSL Headers: ${OPENSSL_INCLUDE_DIR}")
endif()

find_package(Boost COMPONENTS system thread REQUIRED)
find_package(Threads REQUIRED)
find_package(OpenCV REQUIRED)

set(RAPP_LIBRARIES ${RAPP_LIBRARY} 
                   ${OPENSSL_LIBRARIES} 
				   ${Boost_LIBRARIES}
				   ${CMAKE_THREAD_LIBS_INIT})

target_link_libraries(qr_recognition ${RAPP_LIBRARIES}
                                     ${OpenCV_LIBS})

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
