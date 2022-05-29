
add_library(common STATIC
    params.cc
    statlog.cc
    swaglog.cc
    util.cc
    gpio.cc
    i2c.cc
    watchdog.cc
)

target_include_directories(common PUBLIC ${CMAKE_SOURCE_DIR}/third_party/json11)

add_library(gpucommon STATIC
    clutil.cc
    visionimg.cc
)

if(BUILD_TESTING)
    add_executable(test_util tests/test_util.cc ${CMAKE_SOURCE_DIR}/third_party/json11/json11.cpp)
    target_include_directories(test_util PRIVATE ../cereal)
    target_link_libraries(test_util common Catch2::Catch2WithMain)

    add_executable(test_swaglog tests/test_swaglog.cc  ${CMAKE_SOURCE_DIR}/third_party/json11/json11.cpp)
    target_include_directories(test_swaglog PRIVATE ../cereal)
    target_link_libraries(test_swaglog common ${ZMQ_LINK_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT} Catch2::Catch2WithMain)
endif()


# Cython
# envCython.Program('clock.so', 'clock.pyx')
# envCython.Program('params_pyx.so', 'params_pyx.pyx', LIBS=envCython['LIBS'] + [_common, 'zmq', 'json11'])
