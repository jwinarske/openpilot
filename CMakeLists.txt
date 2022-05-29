cmake_minimum_required(VERSION 3.16)

project(selfdrive LANGUAGES CXX C)

set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

find_package(Threads REQUIRED)
find_package(PkgConfig REQUIRED)
pkg_check_modules(ZMQ REQUIRED libzmq)
pkg_check_modules(JPEG REQUIRED libjpeg)
find_package(CapnProto REQUIRED)

include_directories(
    .
    ${CAPNP_INCLUDE_DIRS}
    ${ZMQ_INCLUDE_DIRS}
)

if(BUILD_TESTING)
    find_package(Catch2 REQUIRED)
    include(CTest)
    include(Catch)
    include_directories(
        ${CMAKE_SOURCE_DIR}/third_party/catch2/include
    )
endif()

add_subdirectory(common)
add_subdirectory(cereal)
add_subdirectory(selfdrive/camerad)