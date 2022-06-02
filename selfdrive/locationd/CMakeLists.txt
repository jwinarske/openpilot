
#
# Kalman Filter library
#
find_program(PYTHON3 python3 REQUIRED)
set(GEN_DIR ${CMAKE_CURRENT_SOURCE_DIR}/models/generated)
set(PYTHONPATH "PYTHONPATH=rednose/helpers:selfdrive/locationd/models:")
file(MAKE_DIRECTORY ${GEN_DIR})
macro(generate_kf model python_file)
    set(EXTRA_ARTIFACT)
    if(${model} STREQUAL "live")
        set(EXTRA_ARTIFACT ${GEN_DIR}/${model}_kf_constants.h)
    endif()
    add_custom_command(
        OUTPUT ${GEN_DIR}/${model}.cpp ${GEN_DIR}/${model}.h ${EXTRA_ARTIFACT}
        COMMAND ${CMAKE_COMMAND} -E env ${PYTHONPATH} ${PYTHON3} ${python_file} ${model} ${GEN_DIR}
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
        COMMENT "Generating ${model}_kf"
        VERBATIM
    )
    add_custom_target(${model}_kf
       DEPENDS ${GEN_DIR}/${model}.cpp ${GEN_DIR}/${model}.h ${EXTRA_ARTIFACT}
    )
endmacro()

generate_kf(live selfdrive/locationd/models/live_kf.py)
generate_kf(car selfdrive/locationd/models/car_kf.py)
if(NOT ${TARGET_ARCH} STREQUAL "larch64")
    generate_kf(gnss selfdrive/locationd/models/gnss_kf.py)
    generate_kf(loc_4 selfdrive/locationd/models/loc_kf.py)
    generate_kf(pos_computer_4 rednose/helpers/lst_sq_computer.py)
    generate_kf(pos_computer_5 rednose/helpers/lst_sq_computer.py)
    generate_kf(feature_handler_5 rednose/helpers/feature_handler.py)
endif()

set(KF_SRC
    ${CMAKE_SOURCE_DIR}/rednose/helpers/common_ekf.cc
    models/generated/live.cpp
    models/generated/car.cpp
)

if(NOT ${TARGET_ARCH} STREQUAL "larch64")
    list(APPEND KF_SRC
	models/generated/feature_handler_5.cpp
        models/generated/gnss.cpp
        models/generated/loc_4.cpp
        models/generated/pos_computer_4.cpp
        models/generated/pos_computer_5.cpp)
endif()

add_library(kf STATIC ${KF_SRC})
target_include_directories(kf PRIVATE
    ${CMAKE_SOURCE_DIR}/rednose/helpers
    models/generated
)
add_dependencies(kf live_kf car_kf)
if(NOT ${TARGET_ARCH} STREQUAL "larch64")
    add_dependencies(kf gnss_kf loc_4_kf pos_computer_4_kf pos_computer_5_kf feature_handler_5_kf)
endif()

#
# Kaitai compiler
#
if (BUILD_KAITAI)
    find_package(kaitai-struct-compiler REQUIRED)
    set(GENERATED_DIR "${CMAKE_CURRENT_SOURCE_DIR}/generated")
    set(KAITAI_CMD "kaitai-struct-compiler --target cpp_stl --outdir ${GENERATED_DIR}")
    add_custom_command(
        OUTPUT generated/ubx.cpp generated/ubx.h
        COMMAND kaitai-struct-compiler --target cpp_stl --outdir ${GENERATED_DIR} ubx.ksy
        BYPRODUCTS generated/ubx.cpp generated/ubx.h
        VERBATIM)
    add_custom_command(
        OUTPUT generated/gps.cpp generated/gps.h
        COMMAND kaitai-struct-compiler --target cpp_stl --outdir ${GENERATED_DIR} gps.ksy
        BYPRODUCTS generated/ubx.cpp generated/ubx.h
        VERBATIM)
endif()

add_executable(ubloxd
    ubloxd.cc
    ublox_msg.cc
    generated/ubx.cpp
    generated/gps.cpp
)

target_compile_definitions(ubloxd PRIVATE KS_STR_ENCODING_NONE)
target_include_directories(ubloxd PUBLIC ${CMAKE_SOURCE_DIR}/third_party)

target_link_libraries(ubloxd PRIVATE
    cereal
    messaging
    common
    json11
    kaitai
    ${ZMQ_LINK_LIBRARIES}
    CapnProto::capnp CapnProto::kj
    ${CMAKE_THREAD_LIBS_INIT}
)

add_executable(locationd
    locationd.cc
    models/live_kf.cc
    ${CMAKE_SOURCE_DIR}/rednose/helpers/ekf_sym.cc
)

target_include_directories(locationd PRIVATE generated ${CMAKE_SOURCE_DIR}/cereal)

set(LOC_LIBS
    cereal
    messaging
    common
    transformations
    json11
    kf
    ${ZMQ_LINK_LIBRARIES}
    CapnProto::capnp CapnProto::kj
    ${CMAKE_THREAD_LIBS_INIT}
)

target_link_libraries(locationd PRIVATE ${LOC_LIBS})

if (EXISTS liblocationd.cc)
    add_library(liblocationd SHARED liblocationd.cc)
    target_link_libraries(liblocationd PRIVATE ${LOC_LIBS})
    set_target_properties(liblocationd PROPERTIES OUTPUT_NAME "locationd")
endif()
