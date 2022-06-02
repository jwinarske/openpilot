
set(SRC_FILES main.cc
    cameras/camera_common.cc
    transforms/rgb_to_yuv.cc
    imgproc/utils.cc
)
if (BUILD_QCOM)
    list(APPEND SRC_FILES cameras/camera_qcom2.cc)
endif()

add_executable(camerad ${SRC_FILES})

target_include_directories(camerad PUBLIC 
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${ZMQ_INCLUDE_DIRS}
    ${CMAKE_SOURCE_DIR}/third_party/libyuv/include
    )

target_link_libraries(camerad PUBLIC
    common gpucommon
    cereal messaging
    json11 visionipc
    CapnProto::capnp CapnProto::kj
    ${CMAKE_THREAD_LIBS_INIT}
    ${JPEG_LINK_LIBRARIES}
    ${ZMQ_LINK_LIBRARIES}
    yuv
    m
)

if (BUILD_QCOM)
    target_link_libraries(camerad PUBLIC atomic)
endif()

if (APPLE)
    target_link_libraries(camerad PUBLIC "-framework OpenCL")
else()
    target_link_libraries(camerad PUBLIC OpenCL)
endif()


if(BUILD_TESTING)
    add_executable(ae_gray_test
        test/ae_gray_test.cc
        cameras/camera_common.cc
        transforms/rgb_to_yuv.cc
    )
    target_include_directories(ae_gray_test PRIVATE
        ${CMAKE_SOURCE_DIR}/cereal
        ${CMAKE_SOURCE_DIR}/third_party/libyuv/include
        )

    target_link_libraries(ae_gray_test PUBLIC 
        common
        gpucommon
        visionipc
        json11
        CapnProto::capnp
        ${JPEG_LINK_LIBRARIES}
        ${ZMQ_LINK_LIBRARIES}
        yuv
        ${CMAKE_THREAD_LIBS_INIT}
        )

    if (APPLE)
        target_link_libraries(ae_gray_test PUBLIC "-framework OpenCL")
    else()
        target_link_libraries(ae_gray_test PUBLIC OpenCL)
    endif()

    add_test(NAME ae_gray_test COMMAND ae_gray_test) 
endif()
