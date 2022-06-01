add_library(json11 STATIC json11/json11.cpp)
target_include_directories(json11 INTERFACE json11)

add_library(kaitai STATIC kaitai/kaitaistream.cpp)
target_include_directories(kaitai PUBLIC .)
target_compile_definitions(kaitai PUBLIC KS_STR_ENCODING_NONE)

include(ExternalProject)
ExternalProject_Add(yuv_external
    GIT_REPOSITORY https://chromium.googlesource.com/libyuv/libyuv
    GIT_TAG main
    SOURCE_DIR ${CMAKE_BINARY_DIR}/libyuv
    UPDATE_COMMAND ""
    BUILD_IN_SOURCE 0
    CMAKE_ARGS
        -DCMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DCMAKE_VERBOSE_MAKEFILE=${CMAKE_VERBOSE_MAKEFILE}
        -DCMAKE_STAGING_PREFIX=${CMAKE_BINARY_DIR}/out/usr
        -DCMAKE_POSITION_INDEPENDENT_CODE=ON
    )
add_library(yuv INTERFACE)
target_include_directories(yuv INTERFACE ${CMAKE_BINARY_DIR}/out/usr/include)
target_link_libraries(yuv INTERFACE ${CMAKE_BINARY_DIR}/third_party/yuv_external-prefix/src/yuv_external-build/libyuv.a)
add_dependencies(yuv yuv_external)

add_library(snpe INTERFACE)
target_include_directories(snpe INTERFACE ${CMAKE_SOURCE_DIR}/third_party/snpe/include)
if(APPLE)
    target_link_libraries(snpe INTERFACE ${CMAKE_SOURCE_DIR}/third_party/snpe/larch64/libSNPE.so)
else()
    target_link_libraries(snpe INTERFACE ${CMAKE_SOURCE_DIR}/third_party/snpe/x86_64/libSNPE.so)
endif()
