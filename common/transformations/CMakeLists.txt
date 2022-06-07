
add_library(transformations STATIC orientation.cc coordinates.cc)

add_cython_target(common.transformations.transformations_pyx transformations.pyx CXX)
add_library(common.transformations.transformations_pyx MODULE ${common.transformations.transformations_pyx})
set_target_properties(common.transformations.transformations_pyx PROPERTIES PREFIX "" OUTPUT_NAME "transformations_pyx")
target_include_directories(common.transformations.transformations_pyx PRIVATE ${Python3_INCLUDE_DIRS} ${Python3_NumPy_INCLUDE_DIRS} ${CMAKE_CURRENT_SOURCE_DIR})
target_link_libraries(common.transformations.transformations_pyx PRIVATE ${Python3_LIBRARIES})
