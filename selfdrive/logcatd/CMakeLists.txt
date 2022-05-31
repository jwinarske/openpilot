if(SYSTEMD_FOUND)
    add_executable(logcatd
        logcatd_systemd.cc
    )

    target_link_libraries(logcatd PRIVATE
        CapnProto::capnp CapnProto::kj
        cereal common messaging json11
        ${ZMQ_LINK_LIBRARIES}
        ${SYSTEMD_LINK_LIBRARIES}
    )
endif()