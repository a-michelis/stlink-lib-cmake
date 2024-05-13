# StLink-Lib-CMake : Project Usage

This fork of the original project is patched in such a way that its installation produces a 
cmake _sub-project_ that can be used with `add_subdirectory` from other projects. 

In more detail, installing the project in a directory will produce the following directory structure:


### directory `bin`

- MSVC: This directory contains all built shared libraries (`.dll`).
- UNIX: This directory is not being generated, or is left empty.

### directory `lib`

- MSVC: This directory contains all static libraries (`.lib`), as well as the shared libraries' import libraries (also `.lib`).
- UNIX: This directory contains all shared (`.so`) and static (`.a`) libraries

### directory `include`

This directory contains all project's public header files

### directory `chips`

This directory contains all `.chip` files that are being used for initializing stlink library.

### file `CMakeLists.txt`

This file contains all the import logic used by CMake to make the project available for linkage with other targets.
This CMake project is to be used with `add_subdirectory` in other projects, allowing them to utilize stlink library.

# Building

To differenciate between static and shared libraries, this project allows either full-static or full-shared library creation.
This is dictated via the CMAKE VARIABLE `BUILD_SHARED_LIBS`.

**Static Libraries:**
```shell
cmake $STLINK_LIB_CMAKE_ROOT \
      -DCMAKE_BUILD_TYPE=Release \
      --install-prefix "/the/path/where/the/generated/project/should/exist"

```

**Shared Libraries:**
```shell
cmake $STLINK_LIB_CMAKE_ROOT \
      -DCMAKE_BUILD_TYPE=Release \
      -DBUILD_SHARED_LIBS=ON \
      --install-prefix "/the/path/where/the/generated/project/should/exist"
```

> Remember to replace `$STLINK_LIB_CMAKE_ROOT` with the path of the cloned project's root
