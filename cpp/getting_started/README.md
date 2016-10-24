#Getting Started with RAPP API 
---------------------

Version RAPP API: 0.7.0.

*These instructions are for Linux/Unix based operative systems.*

##Packages required

Before installing RAPP API you need to install these packages:

- GCC and G++ 4.9 or higher
- Boost 1.49 or higher  
- CMake 2.6 or higher 
- Git

The packages can be installed using a terminal and the following commands:

```shell
sudo apt-get install gcc g++ git cmake libboost-all-dev
``` 

##Download

You can download directly from git in this [link](https://github.com/rapp-project/rapp-api.git) with your web browser or use the next command in your terminal:

```shell
git clone https://github.com/rapp-project/rapp-api.git
```

This command download RAPP API in the directory where you are working.

## Building

For the C++ RAPP API, please note you need to satisfy these [*dependencies*](#packages required).
To build:

```shell
cd rapp-api/cpp
mkdir build
cd build
cmake ..
make
```

You should end up with a library and/or tests and examples under your build directory.

## CMake Options

*Note* that you can pass a variety of CMake arguments to suit your needs:

* `-DRAPP_SHARED=ON`                                (Default: compile a shared librapp)
* `-DRAPP_STATIC=ON`                                (Optional: compile a static librapp)
* `-DBUILD_TESTS=ON`                                (Optional: compile various tests - run with `make test`)
* `-DBUILD_EXAMPLES=ON`                             (Optional: compile various examples)
* `-DCMAKE_BUILD_TYPE=Debug`                        (Optional: enable debug symbols)
* `-DSTATIC_LIBSTD=ON`                              (Optional: compile statically against libstdc++)
* `-DOPEN_NAO=ON`                                   (Optional: compiling on OpenNAO requires additional CXX flags)
* `-DCMAKE_INSTALL_PREFIX=/usr`                     (Optional: specify the library install directory
* `-DOPENSSL_ROOT_DIR=/usr/local/opt/openssl`       (Optional: specify OpenSSL root directory)
* `-DOPENSSL_LIBRARIES=/usr/local/opt/openssl/lib/` (Optional: specify OpenSSL library directory)

All of those arguments are optional, however depending on your scenario you may have to adjust.
For example, building for a NAO robot the convenience flag `-DOPEN_NAO=ON` will create a static library optimising for an ATOM cpu
using i386 architecture.

## Installing

You may install if you wish to (as root or using `sudo`):

```shell
make install
```

The produced library will be installed in `/usr/local/lib/` whereas the headers are installed in `/usr/local/include/` by default.

If you wish to install to another location or a user directory then run:

```shell
-DCMAKE_INSTALL_PREFIX=/other/dir
```

*Bear in mind* that this only changes the library installation location and **not** the header install location.

## Testing

Enable tests when you run cmake with:

```shell
cmake .. -DBUILD_TESTS=ON
```

This will create a `rapp-api/cpp/build/tests` directory for each file in `rapp-api/cpp/tests`.
There are four types of tests:

* JSON (de)serialisation tests.
* Class unit tests.
* Cloud functionality tests.
* ASIO unit tests.

To execute them, type:

```shell
make test
```

*_Warning_*: do not run individual tests from within the `/rapp-api/cpp/build/tests` directory!

*_Note_*: all examples and test have been checked with `valgrind --leak-check=full` under Ubuntu 14.04 and found
to have no memory leaks or segfaults.

If you do happen to run across such issues, please open an issue on GitHub.

## Examples

The source files for the examples are under `/rapp-api/cpp/examples` and if you use the CMake option flag `-DBUILD_EXAMPLES=ON` they
will be built under `/rapp-api/cpp/build/examples/`.

You can run them individually, or use them as templates.
