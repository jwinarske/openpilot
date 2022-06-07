
set(SRC_FILES
    sensors/file_sensor.cc
    sensors/i2c_sensor.cc
    sensors/light_sensor.cc
    sensors/bmx055_accel.cc
    sensors/bmx055_gyro.cc
    sensors/bmx055_magn.cc
    sensors/bmx055_temp.cc
    sensors/lsm6ds3_accel.cc
    sensors/lsm6ds3_gyro.cc
    sensors/lsm6ds3_temp.cc
    sensors/mmc5603nj_magn.cc
)

set(LIBS
    common
    cereal
    json11
    messaging
    CapnProto::capnp CapnProto::kj
    ${ZMQ_LINK_LIBRARIES}
)

if (${TARGET_ARCH} STREQUAL "larch64")
    list(APPEND LIBS i2c)
endif()

add_executable(_sensord sensors_qcom2.cc ${SRC_FILES})
target_link_libraries(_sensord PRIVATE ${LIBS})
add_ipo(_sensord)

