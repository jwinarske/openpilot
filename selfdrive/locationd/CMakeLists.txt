
# Import('env', 'common', 'cereal', 'messaging', 'libkf', 'transformations')

# loc_libs = [cereal, messaging, 'zmq', common, 'capnp', 'kj', 'kaitai', 'pthread']


# if GetOption('kaitai'):
#   generated = Dir('generated').srcnode().abspath
#   cmd = f"kaitai-struct-compiler --target cpp_stl --outdir {generated} $SOURCES"
#   env.Command(['generated/ubx.cpp', 'generated/ubx.h'], 'ubx.ksy', cmd)
#   env.Command(['generated/gps.cpp', 'generated/gps.h'], 'gps.ksy', cmd)

# env.Program("ubloxd", ["ubloxd.cc", "ublox_msg.cc", "generated/ubx.cpp", "generated/gps.cpp"], LIBS=loc_libs)

add_executable(ubloxd
    ubloxd.cc
    ublox_msg.cc
    generated/ubx.cpp
    generated/gps.cpp
)

target_compile_definitions(ubloxd PRIVATE KS_STR_ENCODING_NONE)
target_include_directories(ubloxd PUBLIC
    ${CMAKE_SOURCE_DIR}/third_party
)

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
# add_dependencies(locationd REDNOSE_CONFIG_LIVE)

target_include_directories(locationd PRIVATE generated ${CMAKE_SOURCE_DIR}/cereal)

target_link_libraries(locationd PRIVATE 
    cereal
    messaging
    common
    transformations
    ${ZMQ_LINK_LIBRARIES}
    CapnProto::capnp CapnProto::kj
    ${CMAKE_THREAD_LIBS_INIT}
)

#ekf_sym_cc = env.SharedObject("#rednose/helpers/ekf_sym.cc")
#locationd_sources = ["locationd.cc", "models/live_kf.cc", ekf_sym_cc]
#lenv = env.Clone()
#lenv["_LIBFLAGS"] += f' {libkf[0].get_labspath()}'
#locationd = lenv.Program("locationd", locationd_sources, LIBS=loc_libs + transformations)
#lenv.Depends(locationd, libkf)


add_library(liblocationd SHARED liblocationd.cc)
target_link_libraries(liblocationd PRIVATE 
    cereal
    messaging
    common
    transformations
    ${ZMQ_LINK_LIBRARIES}
    CapnProto::capnp CapnProto::kj
    ${CMAKE_THREAD_LIBS_INIT}
)

#if File("liblocationd.cc").exists():
#  liblocationd = lenv.SharedLibrary("liblocationd", ["liblocationd.cc"] + locationd_sources, LIBS=loc_libs + transformations)
#  lenv.Depends(liblocationd, libkf)
