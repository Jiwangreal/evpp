cmake_minimum_required (VERSION 2.8)


include_directories(${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/boost-1.82/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/libgoogle-glog-dev_0.4.0/usr/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/libgflags-dev_2.2.2/usr/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/libevent-dev_2.1.11-stable-1_amd64/usr/include/)

link_directories(${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/libevent-dev_2.1.11-stable-1_amd64/usr/lib/x86_64-linux-gnu)

find_library(Boost_SYSTEM_LIBRARY NAMES boost_system HINTS ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/boost-1.82/lib)
find_library(Boost_THREAD_LIBRARY NAMES boost_thread HINTS ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/boost-1.82/lib)
find_library(GLOG_LIBRARY NAMES glog HINTS ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/libgoogle-glog-dev_0.4.0/usr/lib/x86_64-linux-gnu)
find_library(GFLAGS_LIBRARY NAMES gflags HINTS ${CMAKE_CURRENT_SOURCE_DIR}/3rdparty/libgflags-dev_2.2.2/usr/lib/x86_64-linux-gnu/)


project (safe-evpp C CXX)

if (UNIX)
    if(NOT CMAKE_BUILD_TYPE)
        set(CMAKE_BUILD_TYPE "Release")
    endif()

    set (CXX_FLAGS
         -g
         -fPIC
         -Wall
         -Wno-unused-parameter
         -Wno-unused-function
         -Wunused-variable
         -Wunused-value
         -Wshadow
         -Wcast-qual
         -Wcast-align
         -Wwrite-strings
         -Wsign-compare
         -Winvalid-pch
         -fms-extensions
         -Wfloat-equal
         -Wextra
         -std=c++11
    )
    include_directories (${PROJECT_SOURCE_DIR}
                         ${PROJECT_SOURCE_DIR}/3rdparty)
else (UNIX)
set (CXX_FLAGS -DWIN32 -D_WIN32 -EHsc)
    set(VcpkgRoot ${_VCPKG_INSTALLED_DIR}/${VCPKG_TARGET_TRIPLET})
    include_directories(${VcpkgRoot}/include
                        ${PROJECT_SOURCE_DIR}
                        ${PROJECT_SOURCE_DIR}/3rdparty/wininclude)

    # On windows, Debug mode
    link_directories(${VcpkgRoot}/debug/lib ${VcpkgRoot}/debug/lib/manual-link)

    # On Windows, Release mode
    # link_directories(${VcpkgRoot}/lib ${VcpkgRoot}/lib/manual-link)
endif (UNIX)

string (REPLACE ";" " " CMAKE_CXX_FLAGS "${CXX_FLAGS}")

if (UNIX)
    SET(CMAKE_CXX_FLAGS_DEBUG   "-O0 -g -ggdb -D_DEBUG -DGOOGLE_STRIP_LOG=0")
    SET(CMAKE_CXX_FLAGS_RELEASE "-O3 -g -ggdb -DNDEBUG -DGOOGLE_STRIP_LOG=0")
    SET(DEPENDENT_LIBRARIES event ${GLOG_LIBRARY} ${GFLAGS_LIBRARY} pthread)
else (UNIX)
    SET(DEPENDENT_LIBRARIES event ${GLOG_LIBRARY})
endif (UNIX)

# Set to true if https support is needed.
# Note that this needs openssl
# SET(HTTPS True)
if (HTTPS)
    list(APPEND DEPENDENT_LIBRARIES ssl crypto event_openssl)
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DEVPP_HTTP_CLIENT_SUPPORTS_SSL")
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DEVPP_HTTP_SERVER_SUPPORTS_SSL")
endif (HTTPS)


if (CMAKE_BENCHMARK_TESTING)
	SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DH_BENCHMARK_TESTING=1")
endif (CMAKE_BENCHMARK_TESTING)

set (EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
set (LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)

#set (MACOSX_RPATH ${LIBRARY_OUTPUT_PATH})
set (CMAKE_MACOSX_RPATH 0)

find_program(THRIFT_COMPILER thrift)
find_path(THRIFT_INCLUDE_DIR thrift)
find_library(THRIFT_LIBRARY NAMES thrift)

message (STATUS "This is BINARY dir " ${PROJECT_BINARY_DIR})
message (STATUS "This is SOURCE dir " ${PROJECT_SOURCE_DIR})
message (STATUS "CMAKE_CXX_FLAGS " ${CMAKE_CXX_FLAGS})
message (STATUS "EXECUTABLE_OUTPUT_PATH " ${EXECUTABLE_OUTPUT_PATH})
message (STATUS "LIBRARY_OUTPUT_PATH " ${LIBRARY_OUTPUT_PATH})
message (STATUS "THRIFT_COMPILER " ${THRIFT_COMPILER})
message (STATUS "THRIFT_INCLUDE_DIR " ${THRIFT_INCLUDE_DIR})
message (STATUS "THRIFT_LIBRARY " ${THRIFT_LIBRARY})

add_subdirectory (evpp)
add_subdirectory (apps)
add_subdirectory (labs)

if (NOT EVPP_VCPKG_BUILD)
    add_subdirectory (test)
    add_subdirectory (examples)
    add_subdirectory (benchmark)
endif ()

set (CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake")
include (utils)
include (packages)

include (CPack)
include (CTest)
