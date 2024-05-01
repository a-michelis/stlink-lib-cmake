###
# General cmake settings
###

cmake_minimum_required(VERSION 3.10.2)
cmake_policy(SET CMP0042 NEW)
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/modules)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)

###
#
# Default cmake directories:
#
# | Target Type             | GNUInstallDirs Variable        | Built-In Default      |
# | ---                     | ---                            | ---                   |
# | RUNTIME                 | ${CMAKE_INSTALL_BINDIR}        | bin                   |
# | LIBRARY                 | ${CMAKE_INSTALL_LIBDIR}        | lib                   |
# | ARCHIVE                 | ${CMAKE_INSTALL_LIBDIR}        | lib                   |
# | PRIVATE_HEADER          | ${CMAKE_INSTALL_INCLUDEDIR}    | include               |
# | PUBLIC_HEADER           | ${CMAKE_INSTALL_INCLUDEDIR}    | include               |
# | FILE_SET (type HEADERS) | ${CMAKE_INSTALL_INCLUDEDIR}    | include               |
#
# | TYPE Argument           | GNUInstallDirs Variable        | Built-In Default      |
# | ---                     | ---                            | ---                   |
# | BIN                     | ${CMAKE_INSTALL_BINDIR}        | bin                   |
# | SBIN                    | ${CMAKE_INSTALL_SBINDIR}       | sbin                  |
# | LIB                     | ${CMAKE_INSTALL_LIBDIR}        | lib                   |
# | INCLUDE                 | ${CMAKE_INSTALL_INCLUDEDIR}    | include               |
# | SYSCONF                 | ${CMAKE_INSTALL_SYSCONFDIR}    | etc                   |
# | SHAREDSTATE             | ${CMAKE_INSTALL_SHARESTATEDIR} | com                   |
# | LOCALSTATE              | ${CMAKE_INSTALL_LOCALSTATEDIR} | var                   |
# | RUNSTATE                | ${CMAKE_INSTALL_RUNSTATEDIR}   | <LOCALSTATE dir>/run  |
# | DATA                    | ${CMAKE_INSTALL_DATADIR}       | <DATAROOT dir>        |
# | INFO                    | ${CMAKE_INSTALL_INFODIR}       | <DATAROOT dir>/info   |
# | LOCALE                  | ${CMAKE_INSTALL_LOCALEDIR}     | <DATAROOT dir>/locale |
# | MAN                     | ${CMAKE_INSTALL_MANDIR}        | <DATAROOT dir>/man    |
# | DOC                     | ${CMAKE_INSTALL_DOCDIR}        | <DATAROOT dir>/doc    |
#
# ${CMAKE_BINARY_DIR}
# This is the full path to the top level of the current CMake build tree.
# For an in-source build, this would be the same as CMAKE_SOURCE_DIR.
#
# ${CMAKE_SOURCE_DIR}
# This is the full path to the top level of the current CMake source tree.
# For an in-source build, this would be the same as CMAKE_BINARY_DIR.
#
# ${CMAKE_CURRENT_BINARY_DIR}
# The path to the binary directory currently being processed.
# This is the full path to the build directory that is currently being processed by cmake.
# Each directory added by add_subdirectory() will create a binary directory in the build tree,
# and as it is being processed this variable will be set.
# For in-source builds this is the current source directory being processed.
#
# ${CMAKE_CURRENT_SOURCE_DIR}
# The path to the source directory currently being processed.
# This is the full path to the source directory that is currently being processed by cmake.
#
###


###
# General Project Settings
###

project(stlink C)
set(PROJECT_DESCRIPTION "Open source version of the STMicroelectronics ST-LINK Tools")



set(GEN_LIB_TYPE               "")
set(GEN_LIBUSB_LOC             "")
set(GEN_LIBUSB_IMP             "")
set(GEN_STLINK_LOC             "")
set(GEN_STLINK_IMP             "")
set(GEN_COMPILE_DEFS           "")

set(CMAKE_INSTALL_INCLUDEDIR ${CMAKE_INSTALL_PREFIX}/include)

if(BUILD_SHARED_LIBS)
    set(LIBSTLINK_BUILD_SHARED_LIBS_DEFAULT ON)
else()
    set(LIBSTLINK_BUILD_SHARED_LIBS_DEFAULT OFF)
endif()

option(LIBSTLINK_BUILD_SHARED_LIBS "Build Shared Libraries for st-link library" ${LIBSTLINK_BUILD_SHARED_LIBS_DEFAULT})


## Set C build flags
if (NOT MSVC)
    include(${CMAKE_MODULE_PATH}/c_flags.cmake)
else ()
    message(STATUS "MSVC C Flags override to /MT")
    set(CMAKE_C_FLAGS_DEBUG_INIT          "/D_DEBUG /MTd /Zi /Ob0 /Od /RTC1")
    set(CMAKE_C_FLAGS_MINSIZEREL_INIT     "/MT /O1 /Ob1 /D NDEBUG")
    set(CMAKE_C_FLAGS_RELEASE_INIT        "/MT /O2 /Ob2 /D NDEBUG")
    set(CMAKE_C_FLAGS_RELWITHDEBINFO_INIT "/MT /Zi /O2 /Ob1 /D NDEBUG")
endif()


###
# Dependencies
###
if (NOT WIN32)
    set(LIBUSB_ENABLE_UDEV OFF)
endif ()
set(LIBUSB_BUILD_SHARED_LIBS OFF)
add_subdirectory(${PROJECT_SOURCE_DIR}/ext/libusb)

## Check for system-specific additional header files and libraries

include(CheckIncludeFile)
include(CheckLibraryExists)

CHECK_LIBRARY_EXISTS(ssp __stack_chk_fail "" _stack_chk_fail_exists)
if (_stack_chk_fail_exists)
    if (WIN32)
        set(SSP_LIB -static ssp)
    else ()
        set(SSP_LIB ssp)
    endif()
else ()
    set(SSP_LIB "")
endif()

CHECK_INCLUDE_FILE(sys/time.h STLINK_HAVE_SYS_TIME_H)
if (STLINK_HAVE_SYS_TIME_H)
    add_definitions(-DSTLINK_HAVE_SYS_TIME_H)
    list(APPEND GEN_COMPILE_DEFS -DSTLINK_HAVE_SYS_TIME_H)
#    set(GEN_COMPILE_DEFS "${GEN_COMPILE_DEFS} -DSTLINK_HAVE_SYS_TIME_H")
endif()

CHECK_INCLUDE_FILE(unistd.h STLINK_HAVE_UNISTD_H)
if (STLINK_HAVE_UNISTD_H)
    add_definitions(-DSTLINK_HAVE_UNISTD_H)
    list(APPEND GEN_COMPILE_DEFS -DSTLINK_HAVE_UNISTD_H)
#    set(GEN_COMPILE_DEFS "${GEN_COMPILE_DEFS} -DSTLINK_HAVE_UNISTD_H")
endif()

CHECK_INCLUDE_FILE(dirent.h STLINK_HAVE_DIRENT_H)
if (STLINK_HAVE_DIRENT_H)
    add_definitions(-DSTLINK_HAVE_DIRENT_H)
    list(APPEND GEN_COMPILE_DEFS -DSTLINK_HAVE_DIRENT_H)
#    set(GEN_COMPILE_DEFS "${GEN_COMPILE_DEFS} -DSTLINK_HAVE_DIRENT_H")
endif()

CHECK_INCLUDE_FILE(sys/mman.h -DSTLINK_HAVE_SYS_MMAN_H)
if (STLINK_HAVE_SYS_MMAN_H)
    add_definitions(-DSTLINK_HAVE_SYS_MMAN_H)
    list(APPEND GEN_COMPILE_DEFS -DSTLINK_HAVE_SYS_MMAN_H)
#    set(GEN_COMPILE_DEFS "${GEN_COMPILE_DEFS} -DSTLINK_HAVE_SYS_MMAN_H")
endif()

if (MSVC)
    # Use string.h rather than strings.h and disable annoying warnings
    add_definitions(-DHAVE_STRING_H -D_CRT_SECURE_NO_WARNINGS -D_CRT_NONSTDC_NO_WARNINGS /wd4710)
    list(APPEND GEN_COMPILE_DEFS -DHAVE_STRING_H -D_CRT_SECURE_NO_WARNINGS -D_CRT_NONSTDC_NO_WARNINGS /wd4710)
#    set(GEN_COMPILE_DEFS "${GEN_COMPILE_DEFS} -DHAVE_STRING_H -D_CRT_SECURE_NO_WARNINGS -D_CRT_NONSTDC_NO_WARNINGS /wd4710")
endif()

###
# Main build process
###

## Set installation directory for header files


## Define source FILES for stlink library

set(STLINK_SOURCE
        ${PROJECT_SOURCE_DIR}/src/stlink-lib/calculate.c
        ${PROJECT_SOURCE_DIR}/src/stlink-lib/chipid.c
        ${PROJECT_SOURCE_DIR}/src/stlink-lib/common_flash.c
        ${PROJECT_SOURCE_DIR}/src/stlink-lib/common.c
        ${PROJECT_SOURCE_DIR}/src/stlink-lib/flash_loader.c
        ${PROJECT_SOURCE_DIR}/src/stlink-lib/helper.c
        ${PROJECT_SOURCE_DIR}/src/stlink-lib/logging.c
        ${PROJECT_SOURCE_DIR}/src/stlink-lib/map_file.c
        ${PROJECT_SOURCE_DIR}/src/stlink-lib/lib_md5.c
        ${PROJECT_SOURCE_DIR}/src/stlink-lib/md5.c
        ${PROJECT_SOURCE_DIR}/src/stlink-lib/option_bytes.c
        ${PROJECT_SOURCE_DIR}/src/stlink-lib/read_write.c
        ${PROJECT_SOURCE_DIR}/src/stlink-lib/sg.c
        ${PROJECT_SOURCE_DIR}/src/stlink-lib/usb.c
        )

if (WIN32)
    set(STLINK_SOURCE "${STLINK_SOURCE};${PROJECT_SOURCE_DIR}/src/win32/win32_socket.c")

    if (MSVC)
        # Add drop-in replacement for unistd.h to sources
        set(STLINK_SOURCE "${STLINK_SOURCE};${PROJECT_SOURCE_DIR}/src/win32/getopt/getopt.c")
    endif()

    if (NOT STLINK_HAVE_SYS_MMAN_H)
        set(STLINK_SOURCE "${STLINK_SOURCE};${PROJECT_SOURCE_DIR}/src/win32/mmap.c")
    endif()

    if (NOT STLINK_HAVE_SYS_TIME_H)
        set(STLINK_SOURCE "${STLINK_SOURCE};src/win32/sys_time.c")
    endif()
endif()

### TODO: Static and Shared Variants


if(LIBSTLINK_BUILD_SHARED_LIBS)
    add_library(stlink SHARED ${STLINK_SOURCE})
    if (WIN32)
        set_target_properties(stlink PROPERTIES WINDOWS_EXPORT_ALL_SYMBOLS ON)
    endif ()
else()
    add_library(stlink STATIC ${STLINK_SOURCE})
endif()

target_include_directories(stlink PUBLIC ${PROJECT_SOURCE_DIR}/inc)

file(GLOB STLINK_HEADERS "${CMAKE_SOURCE_DIR}/inc/stlink/*.h")
install(FILES ${STLINK_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/stlink/stlink)

if (WIN32)
    target_include_directories(stlink SYSTEM BEFORE PUBLIC ${PROJECT_SOURCE_DIR}/inc/win32)
    file(GLOB STLINK_WIN32_HEADERS "${CMAKE_SOURCE_DIR}/inc/win32/*.h")
    install(FILES ${STLINK_WIN32_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/stlink_dependencies)
    if (MSVC)
        target_include_directories(stlink SYSTEM BEFORE PUBLIC ${PROJECT_SOURCE_DIR}/inc/unistd)
        target_include_directories(stlink SYSTEM BEFORE PUBLIC ${PROJECT_SOURCE_DIR}/inc/getopt)
        file(GLOB STLINK_UNISTD_HEADERS "${CMAKE_SOURCE_DIR}/inc/unistd/*.h")
        file(GLOB STLINK_GETOPT_HEADERS "${CMAKE_SOURCE_DIR}/inc/getopt/*.h")
        install(FILES ${STLINK_UNISTD_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/stlink_dependencies)
        install(FILES ${STLINK_GETOPT_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/stlink_dependencies)
    endif()

    if (NOT STLINK_HAVE_SYS_TIME_H)
        target_include_directories(stlink SYSTEM BEFORE PUBLIC ${PROJECT_SOURCE_DIR}/inc/sys_time)
        file(GLOB STLINK_SYS_TIME_HEADERS "${CMAKE_SOURCE_DIR}/inc/sys_time/*.h")
        install(FILES ${STLINK_SYS_TIME_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/stlink_dependencies)
    endif()
endif()

if (NOT STLINK_HAVE_SYS_MMAN_H)
    target_include_directories(stlink SYSTEM BEFORE PUBLIC ${PROJECT_SOURCE_DIR}/inc/mmap)
    file(GLOB STLINK_MMAN_HEADERS "${CMAKE_SOURCE_DIR}/inc/mmap/*.h")
    install(FILES ${STLINK_MMAN_HEADERS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/stlink_dependencies)
endif()

set_target_properties(stlink PROPERTIES
                        SOVERSION 1
                        VERSION 1.8.0
                        OUTPUT_NAME stlink
                        )

# Link shared library
if (WIN32)
    target_link_libraries(stlink usb-1.0 ${SSP_LIB} wsock32 ws2_32)
else ()
    target_link_libraries(stlink usb-1.0 ${SSP_LIB})
endif()


install(TARGETS stlink
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR})


if (BUILD_SHARED_LIBS)
    set(GEN_LIB_TYPE "SHARED")
    if (WIN32)
        set(GEN_LIBUSB_LOC "lib/libusb-1.0.lib")
        set(GEN_LIBUSB_IMP "lib/libusb-1.0.lib")
        set(GEN_STLINK_LOC "bin/stlink.dll")
        set(GEN_STLINK_IMP "lib/stlink.lib")
    else ()
        set(GEN_LIBUSB_LOC "lib/libusb-1.0.a")
        set(GEN_LIBUSB_IMP "lib/libusb-1.0.a")
        set(GEN_STLINK_LOC "lib/libstlink.so.1.8.0")
        set(GEN_STLINK_IMP "lib/libstlink.so.1.8.0")
    endif ()
else ()
    set(GEN_LIB_TYPE "STATIC")
    if (WIN32)
        set(GEN_LIBUSB_LOC "lib/libusb-1.0.lib")
        set(GEN_LIBUSB_IMP "lib/libusb-1.0.lib")
        set(GEN_STLINK_LOC "lib/stlink.lib")
        set(GEN_STLINK_IMP "lib/stlink.lib")
    else ()
        set(GEN_LIBUSB_LOC "lib/libusb-1.0.a")
        set(GEN_LIBUSB_IMP "lib/libusb-1.0.a")
        set(GEN_STLINK_LOC "lib/libstlink.a")
        set(GEN_STLINK_IMP "lib/libstlink.a")
    endif ()
endif()

string (REPLACE ";" " " GEN_COMPILE_DEFS "${GEN_COMPILE_DEFS}")
set($ $)
configure_file(
        "${PROJECT_SOURCE_DIR}/importing/CMakeLists.txt.in"
        "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/CMakeLists.txt"
)

install(FILES
        ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/CMakeLists.txt
        DESTINATION
        ${CMAKE_INSTALL_PREFIX}
)

file(GLOB CHIP_FILES RELATIVE ${PROJECT_SOURCE_DIR} "config/chips/*.chip")
install(FILES ${CHIP_FILES} DESTINATION ${CMAKE_INSTALL_PREFIX}/chips)
