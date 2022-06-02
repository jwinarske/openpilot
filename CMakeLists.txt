cmake_minimum_required(VERSION 3.16)

if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build, options are: Debug, Release, or MinSizeRel." FORCE)
    message(STATUS "CMAKE_BUILD_TYPE not set, defaulting to Release.")
endif ()

list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)

if (NOT BUILD_NUMBER)
    set(BUILD_NUMBER 0)
endif ()

project(selfdrive
    VERSION "1.0.${BUILD_NUMBER}"
    LANGUAGES CXX C
    )

include (options)
include (detect_target_arch)
include (get_deps)

set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_POSITION_INDEPENDENT_CODE ON)

include_directories(
    .
    ${CAPNP_INCLUDE_DIRS}
    ${ZMQ_INCLUDE_DIRS}
)

if(BUILD_TESTING)
    find_package(Catch2 REQUIRED)
    include(CTest)
    include(Catch)
    include_directories(${CMAKE_SOURCE_DIR}/third_party/catch2/include)
endif()

if (APPLE)
    add_compile_definitions(GL_SILENCE_DEPRECATION)
endif()

add_subdirectory(common)
add_subdirectory(common/kalman)
add_subdirectory(common/transformations)
add_subdirectory(cereal)
add_subdirectory(opendbc/can)
add_subdirectory(selfdrive/boardd)
add_subdirectory(selfdrive/camerad)
add_subdirectory(selfdrive/clocksd)
add_subdirectory(selfdrive/locationd)
add_subdirectory(selfdrive/logcatd)
add_subdirectory(selfdrive/loggerd)
add_subdirectory(selfdrive/modeld)
add_subdirectory(selfdrive/proclogd)
add_subdirectory(selfdrive/sensord)
add_subdirectory(selfdrive/ui)
add_subdirectory(third_party)
