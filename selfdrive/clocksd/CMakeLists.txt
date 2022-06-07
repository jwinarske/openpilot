
add_executable(clocksd clocksd.cc)
target_link_libraries(clocksd PRIVATE
    CapnProto::capnp CapnProto::kj
    cereal common messaging
    ${ZMQ_LINK_LIBRARIES}
)
add_ipo(clocksd)
