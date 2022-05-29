
set(SRC_FILES main.cc
    cameras/camera_common.cc
    transforms/rgb_to_yuv.cc
    imgproc/utils.cc
)

add_executable(camerad ${SRC_FILES}
    ${CMAKE_SOURCE_DIR}/third_party/json11/json11.cpp
    )

target_include_directories(camerad PUBLIC 
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${ZMQ_INCLUDE_DIRS}
    ${CMAKE_SOURCE_DIR}/third_party/libyuv/include
    )

target_link_libraries(camerad PUBLIC
    common
    gpucommon
    cereal
    messaging
    visionipc
    CapnProto::capnp CapnProto::kj
    ${CMAKE_THREAD_LIBS_INIT}
    ${JPEG_LINK_LIBRARIES}
    ${ZMQ_LINK_LIBRARIES}
    m
)

if (BUILD_QCOM)
    target_link_libraries(camerad PUBLIC atomic)
    list(APPEND SRC_FILES cameras/camera_qcom2.cc)
endif()

if (APPLE)
    target_link_libraries(camerad PUBLIC
        "-framework OpenCL"
        ${CMAKE_SOURCE_DIR}/third_party/libyuv/mac_arm64/lib/libyuv.a
        )
else()
    target_link_libraries(camerad PUBLIC OpenCL
        ${CMAKE_SOURCE_DIR}/third_party/libyuv/x64/lib/libyuv.a
        )
endif()


if(BUILD_TESTING)
    add_executable(ae_gray_test
        test/ae_gray_test.cc
        cameras/camera_common.cc
        transforms/rgb_to_yuv.cc
        ${CMAKE_SOURCE_DIR}/third_party/json11/json11.cpp
    )
    target_include_directories(ae_gray_test PRIVATE
        ${CMAKE_SOURCE_DIR}/cereal
        ${CMAKE_SOURCE_DIR}/third_party/libyuv/include
        )

    target_link_libraries(ae_gray_test PUBLIC 
        common
        gpucommon
        visionipc
        ${ZMQ_LINK_LIBRARIES}
        )

    if (APPLE)
        target_link_libraries(ae_gray_test PUBLIC
            "-framework OpenCL"
            ${CMAKE_SOURCE_DIR}/third_party/libyuv/mac_arm64/lib/libyuv.a
            ${JPEG_LINK_LIBRARIES}
            )
    else()
        target_link_libraries(ae_gray_test PUBLIC OpenCL
            ${CMAKE_SOURCE_DIR}/third_party/libyuv/x64/lib/libyuv.a
            ${JPEG_LINK_LIBRARIES}
            )
    endif()
    
endif()
