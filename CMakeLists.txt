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

if(APPLE AND CMAKE_HOST_APPLE)
    set(Qt5_DIR "/opt/homebrew/opt/qt@5/lib/cmake/Qt5")
    set(OPENSSL_ROOT_DIR "/opt/homebrew/opt/openssl@3")
endif()

find_package(CapnProto REQUIRED)
find_package(Cython REQUIRED)
find_package(Threads REQUIRED)

find_package(PkgConfig REQUIRED)
pkg_check_modules(JPEG REQUIRED libjpeg)
pkg_check_modules(ZMQ REQUIRED libzmq)
pkg_check_modules(FFMPEG REQUIRED libavformat libavcodec libswscale libavutil)
pkg_check_modules(USB REQUIRED libusb-1.0)
pkg_check_modules(Z REQUIRED zlib)
find_package(OpenSSL REQUIRED)
pkg_check_modules(SYSTEMD libsystemd)
pkg_check_modules(OPENCL OpenCL)
if(OPENCL_FOUND)
    add_compile_definitions(CL_TARGET_OPENCL_VERSION=300)
endif()
pkg_check_modules(CURL REQUIRED libcurl)

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
add_subdirectory(selfdrive/logcatd)
add_subdirectory(selfdrive/loggerd)
add_subdirectory(selfdrive/modeld)
add_subdirectory(selfdrive/proclogd)
add_subdirectory(selfdrive/sensord)
add_subdirectory(selfdrive/ui)
add_subdirectory(third_party)


set(REDNOSE_CONFIG_TARGET)
set(REDNOSE_CONFIG_SCRIPT)
set(REDNOSE_CONFIG_ARG1)
set(REDNOSE_CONFIG_ARG2)
macro(RednoseConfig)
    MESSAGE(STATUS "${ARGV0} ${ARGV1} [${ARGV2} ${ARGV3}]")
    list(APPEND REDNOSE_CONFIG_TARGET ${ARGV0})
    list(APPEND REDNOSE_CONFIG_SCRIPT ${ARGV1})
    list(APPEND REDNOSE_CONFIG_ARG1 ${ARGV2})
    list(APPEND REDNOSE_CONFIG_ARG2 ${ARGV3})
endmacro()

RednoseConfig("live" "#selfdrive/locationd/models/live_kf.py" "1" "live_kf_constants.h")
RednoseConfig("car" "#selfdrive/locationd/models/car_kf.py" "1" "")
if (NOT QCOM_BUILD)
    RednoseConfig("gnss" "#selfdrive/locationd/models/gnss_kf.py" "1" "")
    RednoseConfig("loc_4" "#selfdrive/locationd/models/loc_kf.py" "1" "")
    RednoseConfig("pos_computer_4" "#rednose/helpers/lst_sq_computer.py" "0" "")
    RednoseConfig("pos_computer_5" "#rednose/helpers/lst_sq_computer.py" "0" "")
    RednoseConfig("feature_handler_5" "#rednose/helpers/feature_handler.py" "0" "")
    RednoseConfig("lane" "#xx/pipeline/lib/ekf/lane_kf.py" "1" "")
endif()

#add_subdirectory(selfdrive/locationd)
