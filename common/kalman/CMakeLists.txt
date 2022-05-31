add_cython_target(simple_kalman_impl simple_kalman_impl.pyx CXX)
add_library(simple_kalman_impl MODULE ${simple_kalman_impl})
target_include_directories(simple_kalman_impl PRIVATE ${Python3_INCLUDE_DIRS} ${CMAKE_CURRENT_SOURCE_DIR})
target_link_libraries(simple_kalman_impl PRIVATE ${Python3_LIBRARIES})
