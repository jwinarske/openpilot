set(LIBS
    cereal common
    CapnProto::capnp CapnProto::kj
    ${CMAKE_THREAD_LIBS_INIT}
    ${FFMPEG_LINK_LIBRARIES}
    json11 messaging visionipc OpenCL::OpenCL
    ${Z_LINK_LIBRARIES}
    ${ZMQ_LINK_LIBRARIES}
    yuv
    )

set(SRC_FILES
    logger.cc
    video_writer.cc
    encoder/encoder.cc
)

if (${TARGET_ARCH} STREQUAL "larch64")
    list(APPEND SRC_FILES encoder/v4l_encoder.cc)
else()
    list(APPEND SRC_FILES encoder/ffmpeg_encoder.cc)
endif()


add_library(logger SHARED ${SRC_FILES})
target_include_directories(logger PRIVATE libyuv ${FFMPEG_INCLUDE_DIRS})
target_link_libraries(logger PRIVATE ${LIBS})

add_executable(loggerd loggerd.cc)
target_include_directories(loggerd PRIVATE ${FFMPEG_INCLUDE_DIRS})
target_link_libraries(loggerd ${LIBS} logger)
add_ipo(loggerd)

add_executable(encoderd encoderd.cc)
target_include_directories(encoderd PRIVATE ${FFMPEG_INCLUDE_DIRS})
target_link_libraries(encoderd ${LIBS} logger)
add_ipo(encoderd)

add_executable(bootlog bootlog.cc)
target_link_libraries(bootlog ${LIBS} logger)
add_ipo(bootlog)

if (BUILD_TESTING)
    add_executable(test_logger
        tests/test_runner.cc
        tests/test_logger.cc
    )
    target_link_libraries(test_logger PRIVATE
        ${CURL_LINK_LIBRARIES}
        ${CRYPTO_LINK_LIBRARIES}
        ${LIBS}
        logger
    )
    add_test(NAME test_logger COMMAND test_logger)
endif()
