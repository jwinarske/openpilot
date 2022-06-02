
set(LIBS
    cereal messaging
    ${CMAKE_THREAD_LIBS_INIT}
    ${ZMQ_LINK_LIBRARIES}
    CapnProto::capnp CapnProto::kj
    common ${ZMQ_LINK_LIBRARIES} json11
)
add_executable(proclogd main.cc proclog.cc)
target_link_libraries(proclogd PRIVATE ${LIBS})

if (BUILD_TESTING)
    add_executable(test_proclog tests/test_proclog.cc proclog.cc)
    target_link_libraries(test_proclog PRIVATE ${LIBS})
    add_test(NAME test_proclog COMMAND test_proclog)
endif()
