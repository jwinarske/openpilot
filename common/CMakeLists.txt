
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
    add_executable(test_util tests/test_util.cc)
    target_include_directories(test_util PRIVATE ../cereal)
    target_link_libraries(test_util common json11)
    add_test(NAME test_util COMMAND test_util)

    add_executable(test_swaglog tests/test_swaglog.cc)
    target_include_directories(test_swaglog PRIVATE ../cereal)
    target_link_libraries(test_swaglog common json11 ${ZMQ_LINK_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT})
    add_test(NAME test_swaglog COMMAND test_swaglog)
endif()

# Cython
add_cython_target(clock clock.pyx CXX)
add_library(clock MODULE ${clock})
target_include_directories(clock PRIVATE ${Python3_INCLUDE_DIRS})
target_link_libraries(clock ${Python3_LIBRARIES})

add_cython_target(params_pyx clock.pyx CXX)
add_library(params_pyx MODULE ${params_pyx})
target_include_directories(params_pyx PRIVATE ${Python3_INCLUDE_DIRS})
target_link_libraries(params_pyx ${Python3_LIBRARIES} common ${ZMQ_LINK_LIBRARIES} json11)
