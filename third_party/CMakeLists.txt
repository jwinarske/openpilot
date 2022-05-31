add_library(json11 STATIC json11/json11.cpp)
target_include_directories(json11 INTERFACE json11)

add_library(kaitai STATIC kaitai/kaitaistream.cpp)
target_include_directories(kaitai PUBLIC .)
target_compile_definitions(kaitai PUBLIC KS_STR_ENCODING_NONE)

add_subdirectory(libyuv)
target_include_directories(yuv INTERFACE ${CMAKE_SOURCE_DIR}/third_party/libyuv/include)

add_library(snpe INTERFACE)
target_include_directories(snpe INTERFACE ${CMAKE_SOURCE_DIR}/third_party/snpe/include)
if(APPLE)
    target_link_libraries(snpe INTERFACE ${CMAKE_SOURCE_DIR}/third_party/snpe/larch64/libSNPE.so)
else()
    target_link_libraries(snpe INTERFACE ${CMAKE_SOURCE_DIR}/third_party/snpe/x86_64/libSNPE.so)
endif()
