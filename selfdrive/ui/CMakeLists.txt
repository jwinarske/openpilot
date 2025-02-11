set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOMOC ON)
if(APPLE AND CMAKE_HOST_APPLE)
    set(Qt5_DIR "/opt/homebrew/opt/qt@5/lib/cmake/Qt5")
endif()

find_package(Qt5 COMPONENTS Concurrent Core DBus Location Multimedia
    Network OpenGL OpenGLExtensions Qml QuickWidgets Widgets REQUIRED)

set(QT_LIBS Qt5::Core Qt5::Widgets Qt5::Concurrent Qt5::Gui Qt5::Location
            Qt5::Multimedia Qt5::Network Qt5::Qml Qt5::QuickWidgets 
            Qt5::DBus Qt5::OpenGL Qt5::OpenGLExtensions)

set(BASE_LIBS
    gpucommon common messaging cereal visionipc transformations json11
    CapnProto::capnp CapnProto::kj OpenCL::OpenCL OpenSSL::SSL OpenSSL::Crypto 
    ${ZMQ_LINK_LIBRARIES} ${CMAKE_THREAD_LIBS_INIT} m
)

set(MAPS OFF)
if (${TARGET_ARCH} STREQUAL "larch64" OR ${TARGET_ARCH} STREQUAL "x86_64")
    set(MAPS ON)
endif()
#maps = arch in ['larch64', 'x86_64']

if (MAPS AND ${TARGET_ARCH} STREQUAL "x86_64")
#if maps and arch == 'x86_64':
#  rpath = [Dir(f"#third_party/mapbox-gl-native-qt/{arch}").srcnode().abspath]
#  qt_env["RPATH"] += rpath
endif()

set(WIDGETS_SRC
    ui.cc
    qt/util.cc
    qt/widgets/input.cc
    qt/widgets/drive_stats.cc
    qt/widgets/ssh_keys.cc
    qt/widgets/toggle.cc
    qt/widgets/controls.cc
    qt/widgets/offroad_alerts.cc
    qt/widgets/prime.cc
    qt/widgets/keyboard.cc
    qt/widgets/scrollview.cc
    qt/widgets/cameraview.cc
    ${CMAKE_SOURCE_DIR}/third_party/qrcode/QrCode.cc
    qt/api.cc
    qt/request_repeater.cc
    qt/qt_window.cc
    qt/offroad/networking.cc
    qt/offroad/wifiManager.cc
)

if(BUILD_MAPS)
    list(APPEND BASE_LIBS qmapboxgl)
    list(APPEND WIDGETS_SRC qt/maps/map_helpers.cc)
    list(APPEND WIDGETS_SRC qt/maps/map_settings.cc)
    list(APPEND WIDGETS_SRC qt/maps/map.cc)
endif()

add_library(qt_widgets SHARED ${WIDGETS_SRC})
target_include_directories(qt_widgets PRIVATE ${CMAKE_SOURCE_DIR}/third_party/qrcode)
target_link_libraries(qt_widgets PRIVATE ${BASE_LIBS} Qt5::Network Qt5::OpenGL ${QT_LIBS})
list(APPEND QT_LIBS qt_widgets)
list(APPEND QT_LIBS ${BASE_LIBS})
if(BUILD_MAPS)
    target_compile_definitions(qt_widgets PRIVATE ENABLE_MAPS)
endif()

# build assets
qt5_add_resources(MOC_RSRC ${CMAKE_SOURCE_DIR}/selfdrive/assets/assets.qrc)
add_library(asset_obj STATIC ${MOC_RSRC})

# build soundd
add_executable(_soundd soundd/main.cc soundd/sound.cc)
target_link_libraries(_soundd PRIVATE ${QT_LIBS})
add_ipo(_soundd)

if (BUILD_TESTING)
    add_executable(playsound tests/playsound.cc)
    target_link_libraries(playsound PRIVATE ${BASE_LIBS} ${QT_LIBS})
    add_test(NAME playsound COMMAND playsound)
    set_tests_properties(playsound PROPERTIES ENVIRONMENT "QT_QPA_PLATFORM=offscreen")

    add_executable(test_sound tests/test_runner.cc soundd/sound.cc tests/test_sound.cc)
    target_link_libraries(test_sound PRIVATE ${QT_LIBS})
    add_test(NAME test_sound COMMAND test_sound)
endif()

add_library(python_helpers qt/qt_window.cc)
target_link_libraries(python_helpers PRIVATE ${QT_LIBS})
add_ipo(python_helpers)

# spinner and text window
add_executable(text qt/text.cc)
target_link_libraries(text PRIVATE ${QT_LIBS})
add_ipo(text)

add_executable(spinner qt/spinner.cc)
target_link_libraries(spinner PRIVATE ${QT_LIBS})
add_ipo(spinner)

# build main UI
set(QT_SRC
    main.cc
    qt/sidebar.cc
    qt/onroad.cc
    qt/body.cc
    qt/window.cc
    qt/home.cc
    qt/offroad/settings.cc
    qt/offroad/onboarding.cc
    qt/offroad/driverview.cc
)
add_executable(_ui ${QT_SRC})
target_link_libraries(_ui PRIVATE asset_obj ${QT_LIBS})
add_ipo(_ui)

# setup and factory resetter
if (BUILD_EXTRAS)
    add_executable(reset qt/setup/reset.cc)
    target_link_libraries(reset PRIVATE ${QT_LIBS})
    add_ipo(reset)

    add_executable(setup qt/setup/setup.cc)
    target_link_libraries(setup PRIVATE asset_obj ${QT_LIBS} ${CURL_LINK_LIBRARIES} common json11)
    add_ipo(setup)

    # buidl updater UI
    add_executable(updater qt/setup/updater.cc)
    target_link_libraries(updater PRIVATE asset_obj ${QT_LIBS})
    add_ipo(updater)

    # build mui
    add_executable(_mui mui.cc)
    target_link_libraries(_mui PRIVATE ${QT_LIBS})
    add_ipo(_mui)    
endif()


# build headless replay
if (APPLE OR BUILD_EXTRAS)
    set(REPLAY_SRC
        replay/replay.cc
        replay/consoleui.cc
        replay/camera.cc
        replay/filereader.cc
        replay/logreader.cc
        replay/framereader.cc
        replay/route.cc
        replay/util.cc
        )

    add_library(qt_replay STATIC ${REPLAY_SRC})
    target_link_libraries(qt_replay PRIVATE ${BASE_LIBS} ${QT_LIBS})
    target_include_directories(qt_replay PRIVATE
	${CMAKE_BINARY_DIR}/out/usr/include
        ${FFMPEG_INCLUDE_DIRS}
	)
    target_compile_options(qt_replay PRIVATE "-Wno-deprecated-declarations")

    add_executable(replay replay/main.cc)
    set(REPLAY_LIBS qt_replay ${FFMPEG_LINK_LIBRARIES} bz2 ${CURL_LINK_LIBRARIES} yuv ncurses ${QT_LIBS})
    target_include_directories(replay PRIVATE ${FFMPEG_INCLUDE_DIRS})
    target_link_libraries(replay PRIVATE ${REPLAY_LIBS})
    add_ipo(replay)

    add_executable(watch3 watch3.cc)
    target_link_libraries(watch3 PRIVATE ${QT_LIBS} common json11 ${ZMQ_LINK_LIBRARIES} visionipc messaging)
    add_ipo(watch3)

    if (BUILD_TESTING)
        add_executable(test_replay replay/tests/test_runner.cc replay/tests/test_replay.cc)
        target_include_directories(test_replay PRIVATE ${FFMPEG_INCLUDE_DIRS})
        target_link_libraries(test_replay PRIVATE ${REPLAY_LIBS})
        # add_test(NAME test_replay COMMAND test_replay)
    endif()

endif()

# navd
if (BUILD_MAPS)
    add_executable(_navd navd/main.cc navd/route_engine.cc navd/map_renderer.cc)
    target_link_libraries(_navd ${QT_LIBS} common json11)
    add_ipo(_navd)
endif()

if (BUILD_MAPS AND BUILD_EXTRAS)
    add_library(map_renderer SHARED navd/map_renderer.cc)
    target_link_libraries(map_renderer PRIVATE ${QT_LIBS} mapbox-gl-native-qt common messaging)
    add_ipo(map_renderer)
endif()
