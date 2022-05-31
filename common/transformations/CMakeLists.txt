
add_library(transformations STATIC orientation.cc coordinates.cc)

add_cython_target(transformations_pyx transformations.pyx CXX)
add_library(transformations_pyx MODULE ${transformations_pyx})
target_include_directories(transformations_pyx PRIVATE ${Python3_INCLUDE_DIRS} ${Python3_NumPy_INCLUDE_DIRS} ${CMAKE_CURRENT_SOURCE_DIR})
target_link_libraries(transformations_pyx PRIVATE ${Python3_LIBRARIES})
