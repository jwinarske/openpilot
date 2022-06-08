add_library(json11 STATIC json11/json11.cpp)
target_include_directories(json11 INTERFACE json11)

add_library(kaitai STATIC kaitai/kaitaistream.cpp)
target_include_directories(kaitai PUBLIC .)
target_compile_definitions(kaitai PUBLIC KS_STR_ENCODING_NONE)

#
# libyuv
#

include(ExternalProject)
ExternalProject_Add(yuv
    GIT_REPOSITORY https://chromium.googlesource.com/libyuv/libyuv
    GIT_TAG main
    GIT_SHALLOW ON
    SOURCE_DIR ${CMAKE_BINARY_DIR}/libyuv
    UPDATE_COMMAND ""
    BUILD_IN_SOURCE 0
    CMAKE_ARGS
        -DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DCMAKE_VERBOSE_MAKEFILE=${CMAKE_VERBOSE_MAKEFILE}
        -DCMAKE_STAGING_PREFIX=${CMAKE_BINARY_DIR}/out/usr
        -DCMAKE_POSITION_INDEPENDENT_CODE=${CMAKE_POSITION_INDEPENDENT_CODE}
        -DJPEG_FOUND=ON
        -DJPEG_INCLUDE_DIR=${JPEG_INCLUDE_DIR}
        -DJPEG_LIBRARY=${JPEG_LINK_LIBRARIES}
    )
set_target_properties(yuv PROPERTIES
        INCLUDE_DIRECTORIES ${CMAKE_BINARY_DIR}/out/usr/include
        LINK_LIBRARIES ${CMAKE_BINARY_DIR}/out/usr/lib/libyuv.a
        )

#
# SNPE SDK
#

if(DEFINED ENV{SNPE_ROOT})
    set(SNPE_ROOT $ENV{SNPE_ROOT})
    if(NOT DEFINED ENV{SNPE_TARGET_ARCH})
        message(FATAL_ERROR "Set SNPE_TARGET_ARCH to match name of SNPE_ROOT lib subfolder")
    endif()
    set(SNPE_TARGET_ARCH $ENV{SNPE_TARGET_ARCH})
    if(NOT EXISTS ${SNPE_ROOT}/lib/${SNPE_TARGET_ARCH})
        message(FATAL_ERROR "Unknown SNPE_TARGET_ARCH")
    endif()

    set(SNPE_INCLUDE_DIR ${SNPE_ROOT}/include/zdl)
    set(SNPE_LINK_LIBRARIES ${SNPE_ROOT}/lib/${SNPE_TARGET_ARCH}/libSNPE.so)
else()
    if(NOT EXISTS ${CMAKE_SOURCE_DIR}/third_party/snpe/${TARGET_ARCH}/libSNPE.so)
        message(FATAL_ERROR "Unknown TARGET_ARCH")
    endif()

    set(SNPE_INCLUDE_DIR ${CMAKE_SOURCE_DIR}/third_party/snpe/include)
    set(SNPE_LINK_LIBRARIES ${CMAKE_SOURCE_DIR}/third_party/snpe/${TARGET_ARCH}/libSNPE.so)
endif()

add_library(snpe INTERFACE)
target_include_directories(snpe INTERFACE ${SNPE_INCLUDE_DIR})
target_link_libraries(snpe INTERFACE ${SNPE_LINK_LIBRARIES})
