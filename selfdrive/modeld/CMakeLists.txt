
#libs = [cereal, messaging, common, visionipc, gpucommon,
#        'OpenCL', 'SNPE', 'capnp', 'zmq', 'kj', 'yuv']

set(LIBS
    cereal messaging common visionipc gpucommon json11
    ${CMAKE_THREAD_LIBS_INIT} OpenCL snpe
    CapnProto::capnp CapnProto::kj ${ZMQ_LINK_LIBRARIES}
    yuv
)

#def get_dlsym_offset():
#  """Returns the offset between dlopen and dlsym in libdl.so"""
#  import ctypes
#  libdl = ctypes.PyDLL('libdl.so')
#  dlopen = ctypes.cast(libdl.dlopen, ctypes.c_void_p).value
#  dlsym = ctypes.cast(libdl.dlsym, ctypes.c_void_p).value
#  return dlsym - dlopen


set(COMMON_SRC
    models/commonmodel.cc
    runners/snpemodel.cc
    transforms/loadyuv.cc
    transforms/transform.cc
)

set(THNEED_SRC
    thneed/thneed.cc
    thneed/serialize.cc
    thneed/optimizer.cc
    runners/thneedmodel.cc
)

option(BUILD_NO_THNEED "Build without thneed" ON)
#use_thneed = not GetOption('no_thneed')

if (TRUE)
#    list(APPEND LIBS gsl CB dl)

    set(COMMON_MODEL)

    if (NOT BUILD_NO_THNEED)
        list(APPEND COMMON_SRC ${THNEED_SRC})
        add_compile_definitions(USE_THNEED DLSYM_OFFSET=${dlsym_offset})
    endif()

    if (BUILD_SNPE)
        list(APPEND COMMON_SRC runners/onnxmodel.cc)
        add_compile_definitions(USE_ONNX_MODEL)
    endif()

    if(APPLE)
        list(REMOVE_ITEM LIBS OpenCL)
        list(APPEND LIBS "-framework OpenCL")
        list(REMOVE_ITEM LIBS snpe)
        list(REMOVE_ITEM COMMON_SRC runners/snpemodel.cc)
    endif()
endif()

set(COMMON_MODEL ${COMMON_SRC})
# common_model = lenv.Object(common_src)

# build thneed model
#if use_thneed and arch == "larch64":
#  fn = File("models/supercombo").abspath
#  compiler = lenv.Program('thneed/compile', ["thneed/compile.cc"]+common_model, LIBS=libs)
#  cmd = f"cd {Dir('.').abspath} && {compiler[0].abspath} --in {fn}.dlc --out {fn}_badweights.thneed --binary --optimize"

#  lib_paths = ':'.join(Dir(p).abspath for p in lenv["LIBPATH"])
#  kernel_path = os.path.join(Dir('.').abspath, "thneed", "kernels")
#  cenv = Environment(ENV={'LD_LIBRARY_PATH': f"{lib_paths}:{lenv['ENV']['LD_LIBRARY_PATH']}", 'KERNEL_PATH': kernel_path})

#  kernels = [os.path.join(kernel_path, x) for x in os.listdir(kernel_path) if x.endswith(".cl")]
#  cenv.Command(fn + "_badweights.thneed", [fn + ".dlc", kernels, compiler], cmd)

#  from selfdrive.modeld.thneed.weights_fixup import weights_fixup
#  def weights_fixup_action(target, source, env):
#    weights_fixup(target[0].abspath, source[0].abspath, source[1].abspath)

#  env = Environment(BUILDERS = {'WeightFixup' : Builder(action = weights_fixup_action)})
#  env.WeightFixup(target=fn + ".thneed", source=[fn+"_badweights.thneed", fn+".dlc"])


add_executable(_dmonitoringmodeld
    dmonitoringmodeld.cc
    models/dmonitoring.cc
    ${COMMON_MODEL}
)
target_include_directories(_dmonitoringmodeld PRIVATE
    ${CMAKE_SOURCE_DIR}/third_party/libyuv/include
    ${CMAKE_SOURCE_DIR}/third_party/snpe/include
)
target_link_libraries(_dmonitoringmodeld PRIVATE ${LIBS})

add_executable(_modeld
    modeld.cc
    models/driving.cc
    ${COMMON_MODEL}
)
target_include_directories(_modeld PRIVATE
    ${CMAKE_SOURCE_DIR}/third_party/libyuv/include
    ${CMAKE_SOURCE_DIR}/third_party/snpe/include
)
target_link_libraries(_modeld PRIVATE ${LIBS})
