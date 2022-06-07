
set(LIBS
  CapnProto::capnp CapnProto::kj
  cereal common json11 kaitai messaging
  ${CMAKE_THREAD_LIBS_INIT}
  ${USB_LINK_LIBRARIES}
  ${ZMQ_LINK_LIBRARIES}
)

add_executable(boardd 
    main.cc
    boardd.cc
    panda.cc
    pigeon.cc
)
target_link_libraries(boardd PRIVATE ${LIBS})
add_ipo(boardd)

add_library(libcan_list_to_can_capnp STATIC can_list_to_can_capnp.cc)

add_cython_target(selfdrive.boardd.boardd_api_impl boardd_api_impl.pyx CXX)
add_library(selfdrive.boardd.boardd_api_impl MODULE ${selfdrive.boardd.boardd_api_impl})
set_target_properties(selfdrive.boardd.boardd_api_impl PROPERTIES PREFIX "" OUTPUT_NAME "boardd_api_impl")
target_include_directories(selfdrive.boardd.boardd_api_impl PRIVATE ${Python3_INCLUDE_DIRS} ${Python3_NumPy_INCLUDE_DIRS} ${CMAKE_CURRENT_SOURCE_DIR})
target_link_libraries(selfdrive.boardd.boardd_api_impl PRIVATE
    libcan_list_to_can_capnp
    ${Python3_LIBRARIES}
    CapnProto::capnp
    CapnProto::kj
)
add_ipo(selfdrive.boardd.boardd_api_impl)

if (BUILD_TESTING)
  add_executable(test_boardd_usbprotocol
    tests/test_boardd_usbprotocol.cc
    panda.cc
  )
  target_link_libraries(test_boardd_usbprotocol PRIVATE ${LIBS})
  add_test(NAME test_boardd_usbprotocol COMMAND test_boardd_usbprotocol)
endif()
