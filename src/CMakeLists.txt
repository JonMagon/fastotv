SET(SOURCE_ROOT ${CMAKE_SOURCE_DIR}/src)
SET(CLIENT_SOURCE_DIR ${SOURCE_ROOT}/client)
SET(PROJECT_CURRENT_BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME})

IF(OS_WINDOWS)
  SET(RESOURCE_OS     "${PROJECT_CURRENT_BINARY_DIR}/win.rc")
  SET(RESOURCE_OS_IN  "${PROJECT_BRANDING_FOLDER}/windows/winres.rc.in")
  SET(ICON_FILE_IN    "${PROJECT_BRANDING_FOLDER}/windows/icon.ico")
ELSEIF(OS_MACOSX)
  SET(RESOURCE_OS     "${PROJECT_CURRENT_BINARY_DIR}/${PROJECT_NAME_LOWERCASE}.desktop")
  SET(RESOURCE_OS_IN  "${PROJECT_BRANDING_FOLDER}/desktop.in")
  SET(ICON_FILE_IN    "${PROJECT_BRANDING_FOLDER}/macosx/icon.icns")
ELSEIF(OS_LINUX)
  SET(RESOURCE_OS     "${PROJECT_CURRENT_BINARY_DIR}/${PROJECT_NAME_LOWERCASE}.desktop")
  SET(RESOURCE_OS_IN  "${CMAKE_SOURCE_DIR}/install/${PROJECT_NAME_LOWERCASE}/desktop.in")
  SET(ICON_FILE_IN    "${PROJECT_BRANDING_FOLDER}/linux/icon.png")
ELSEIF(OS_FREEBSD)
  SET(RESOURCE_OS     "${PROJECT_CURRENT_BINARY_DIR}/${PROJECT_NAME_LOWERCASE}.desktop")
  SET(RESOURCE_OS_IN  "${PROJECT_BRANDING_FOLDER}/desktop.in")
  SET(ICON_FILE_IN    "${PROJECT_BRANDING_FOLDER}/linux/icon.png")
ELSEIF(OS_ANDROID)
  SET(RESOURCE_OS     "${PROJECT_CURRENT_BINARY_DIR}/${PROJECT_NAME_LOWERCASE}.desktop")
  SET(RESOURCE_OS_IN  "${PROJECT_BRANDING_FOLDER}/desktop.in")
  SET(ICON_FILE_IN    "${PROJECT_BRANDING_FOLDER}/android/icon.png")
ENDIF(OS_WINDOWS)
GET_FILENAME_COMPONENT(ICON_FILE_IN_EXTENSION ${ICON_FILE_IN} EXT)
SET(ICON_FILE ${PROJECT_CURRENT_BINARY_DIR}/${PROJECT_NAME_LOWERCASE}${ICON_FILE_IN_EXTENSION})
CONFIGURE_FILE("${ICON_FILE_IN}" "${ICON_FILE}" COPYONLY)

IF(OS_WINDOWS)
  SET(PLATFORM_HDRS)
  SET(PLATFORM_SRCS)
  SET(PLATFORM_LIBRARIES ws2_32 crypt32 bcrypt)
ELSEIF(OS_MACOSX)
  SET(PLATFORM_HDRS)
  SET(PLATFORM_SRCS)
  SET(PLATFORM_LIBRARIES)
ELSEIF(OS_LINUX)
  SET(PLATFORM_HDRS)
  SET(PLATFORM_SRCS)
  SET(PLATFORM_LIBRARIES)
ELSEIF(OS_FREEBSD)
  SET(PLATFORM_HDRS)
  SET(PLATFORM_SRCS)
  SET(PLATFORM_LIBRARIES)
ENDIF(OS_WINDOWS)

IF(USE_PTHREAD)
  SET(PLATFORM_LIBRARIES ${PLATFORM_LIBRARIES} pthread)
ENDIF(USE_PTHREAD)

# crossplatform libraries
FIND_PACKAGE(CUDA QUIET)
IF(CUDA_FOUND)
  ADD_DEFINITIONS(-DHAVE_CUDA)
  SET(HAVE_CUVID 1)
  SET(CONFIG_CUVID 1)
  SET(DEPENDENS_INCLUDE_DIRS ${DEPENDENS_INCLUDE_DIRS} ${CUDA_INCLUDE_DIRS})
  SET(DEPENDENS_LIBRARIES ${DEPENDENS_LIBRARIES} ${CUDA_LIBRARIES})
ENDIF(CUDA_FOUND)

IF(OS_WINDOWS)
  FIND_PACKAGE(DirectX)
  IF(DirectX_FOUND)
    SET(DXVA2_FOUND ON)
    ADD_DEFINITIONS(-DHAVE_DXVA2)
    SET(HAVE_DXVA2_LIB 1)
    SET(CONFIG_DXVA2 1)
    SET(DEPENDENS_LIBRARIES ${DEPENDENS_LIBRARIES} ${DirectX_LIBRARIES})
    SET(DEPENDENS_INCLUDE_DIRS ${DEPENDENS_INCLUDE_DIRS} ${DIRECTX_INCLUDE_DIRS})
  ENDIF(DirectX_FOUND)
  SET(PLATFORM_HDRS ${PLATFORM_HDRS})
  SET(PLATFORM_SRCS ${PLATFORM_SRCS})
  SET(PLATFORM_LIBRARIES ${PLATFORM_LIBRARIES} secur32 vfw32 strmiids shlwapi)
ELSEIF(OS_MACOSX)
  FIND_LIBRARY(COCOA_LIBRARY Cocoa)
  FIND_LIBRARY(SECURITY_LIBRARY Security)
  FIND_LIBRARY(QTKIT_LIBRARY QTKit)
  FIND_LIBRARY(AVFOUNDATION_LIBRARY AvFoundation)
  FIND_LIBRARY(AUDIOTOOLBOX_LIBRARY AudioToolBox)
  FIND_LIBRARY(FOUNDATION_LIBRARY Foundation)
  FIND_LIBRARY(VIDEODECODE_LIBRARY VideoDecodeAcceleration)
  FIND_LIBRARY(QUARTZ_LIBRARY QuartzCore)
  FIND_LIBRARY(OPENGL_LIBRARY OpenGL)

  FIND_LIBRARY(COREMEDIA_LIBRARY CoreMedia)
  FIND_LIBRARY(COREVIDEO_LIBRARY CoreVideo)
  FIND_LIBRARY(CORESERVICES_LIBRARY CoreServices)
  IF(COREVIDEO_LIBRARY AND COREMEDIA_LIBRARY)
    SET(HAVE_UTGETOSTYPEFROMSTRING 1)
    SET(DEPENDENS_LIBRARIES ${DEPENDENS_LIBRARIES} ${CORESERVICES_LIBRARY})
  ENDIF(COREVIDEO_LIBRARY AND COREMEDIA_LIBRARY)

  FIND_LIBRARY(VIDEOTOOLBOX_LIBRARY VideoToolBox)
  IF (VIDEOTOOLBOX_LIBRARY)
    SET(VIDEOTOOLBOX_FOUND ON)
    SET(HAVE_VIDEOTOOLBOX 1)
    SET(CONFIG_VIDEOTOOLBOX 1)

    SET(DEPENDENS_LIBRARIES ${DEPENDENS_LIBRARIES} ${VIDEOTOOLBOX_LIBRARY})
  ENDIF(VIDEOTOOLBOX_LIBRARY)

  SET(PLATFORM_HDRS ${PLATFORM_HDRS})
  SET(PLATFORM_SRCS ${PLATFORM_SRCS})
  SET(PLATFORM_LIBRARIES
    ${PLATFORM_LIBRARIES}
    ${COCOA_LIBRARY}
    ${SECURITY_LIBRARY}
    ${QTKIT_LIBRARY}
    ${AVFOUNDATION_LIBRARY}
    ${AUDIOTOOLBOX_LIBRARY}
    ${COREVIDEO_LIBRARY}
    ${COREMEDIA_LIBRARY}
    ${FOUNDATION_LIBRARY}
    ${VIDEODECODE_LIBRARY}
    ${QUARTZ_LIBRARY}
    ${OPENGL_LIBRARY}
   )
ELSEIF(OS_LINUX)
  FIND_LIBRARY(VA_DRM_LIBRARY NAMES va-drm DOC "Path to va-drm library")
  IF(VA_DRM_LIBRARY)
    SET(DEPENDENS_LIBRARIES ${DEPENDENS_LIBRARIES} ${VA_DRM_LIBRARY})
  ENDIF(VA_DRM_LIBRARY)
  FIND_LIBRARY(VA_X11_LIBRARY NAMES va-x11 DOC "Path to va-x11 library")
  IF(VA_X11_LIBRARY)
    SET(HAVE_VAAPI_X11 1)
    SET(DEPENDENS_LIBRARIES ${DEPENDENS_LIBRARIES} ${VA_X11_LIBRARY})
  ENDIF(VA_X11_LIBRARY)

  FIND_PACKAGE(PkgConfig REQUIRED)

  FIND_PACKAGE(X11 QUIET)
  IF(X11_FOUND)
    SET(DEPENDENS_LIBRARIES ${DEPENDENS_LIBRARIES} ${X11_LIBRARIES})
    SET(DEPENDENS_INCLUDE_DIRS ${DEPENDENS_INCLUDE_DIRS} ${X11_INCLUDE_DIR})
  ENDIF(X11_FOUND)
  FIND_PACKAGE(ALSA QUIET)
  IF(ALSA_FOUND)
    SET(DEPENDENS_LIBRARIES ${DEPENDENS_LIBRARIES} ${ALSA_LIBRARY})
    SET(DEPENDENS_INCLUDE_DIRS ${DEPENDENS_INCLUDE_DIRS} ${ALSA_INCLUDE_DIR})
  ENDIF(ALSA_FOUND)
  FIND_PACKAGE(Xv QUIET)
  IF(XV_FOUND)
    SET(DEPENDENS_LIBRARIES ${DEPENDENS_LIBRARIES} ${XV_LIBRARIES})
    SET(DEPENDENS_INCLUDE_DIRS ${DEPENDENS_INCLUDE_DIRS} ${XV_INCLUDE_DIRS})
  ENDIF(XV_FOUND)

  # openmax
  FIND_LIBRARY(OPENMAXIL_LIBRARY
    NAMES openmaxil
    DOC "Path to OpenMAX IL library"
    PATHS /opt/vc/lib
  )
  FIND_PATH(OPENMAXIL_INCLUDE_DIR
    NAMES OMX_Core.h
    DOC "Openmax include directory"
    PATHS /opt/vc/include /opt/vc/include/IL
  )
  IF(OPENMAXIL_LIBRARY AND OPENMAXIL_INCLUDE_DIR)
    SET(CONFIG_OMX 1)
    SET(DEPENDENS_LIBRARIES ${DEPENDENS_LIBRARIES} ${OPENMAXIL_LIBRARY})
    SET(DEPENDENS_INCLUDE_DIRS ${DEPENDENS_INCLUDE_DIRS} ${OPENMAXIL_INCLUDE_DIR})
  ENDIF(OPENMAXIL_LIBRARY AND OPENMAXIL_INCLUDE_DIR)

  # broadcom
  FIND_LIBRARY(BCM_HOST_LIBRARY
    NAMES bcm_host
    DOC "Path to Broadcom host library"
    PATHS /opt/vc/lib
  )
  FIND_PATH(BROADCOM_INCLUDE_DIR
    NAMES bcm_host.h
    DOC "Broadcom include directory"
    PATHS /opt/vc/include
  )
  IF(BCM_HOST_LIBRARY AND BROADCOM_INCLUDE_DIR)
    SET(CONFIG_OMX_RPI 1)
    SET(DEPENDENS_LIBRARIES ${DEPENDENS_LIBRARIES} ${BCM_HOST_LIBRARY})
    SET(DEPENDENS_INCLUDE_DIRS ${DEPENDENS_INCLUDE_DIRS} ${BROADCOM_INCLUDE_DIR})

    # broadcom mmal
    FIND_LIBRARY(MMAL_CORE_LIBRARY
      NAMES mmal_core
      DOC "Path to Broadcom mmal core library"
      PATHS /opt/vc/lib
    )
    IF(MMAL_CORE_LIBRARY)
      SET(DEPENDENS_LIBRARIES ${DEPENDENS_LIBRARIES} ${MMAL_CORE_LIBRARY})
    ENDIF(MMAL_CORE_LIBRARY)

    FIND_LIBRARY(MMAL_UTIL_LIBRARY
      NAMES mmal_util
      DOC "Path to Broadcom mmal util library"
      PATHS /opt/vc/lib
    )
    IF(MMAL_UTIL_LIBRARY)
      SET(DEPENDENS_LIBRARIES ${DEPENDENS_LIBRARIES} ${MMAL_UTIL_LIBRARY})
    ENDIF(MMAL_UTIL_LIBRARY)

    FIND_LIBRARY(MMAL_VC_CLIENT_LIBRARY
      NAMES mmal_vc_client
      DOC "Path to Broadcom mmal vc client library"
      PATHS /opt/vc/lib
    )
    IF(MMAL_VC_CLIENT_LIBRARY)
      SET(DEPENDENS_LIBRARIES ${DEPENDENS_LIBRARIES} ${MMAL_VC_CLIENT_LIBRARY})
    ENDIF(MMAL_VC_CLIENT_LIBRARY)
  ENDIF(BCM_HOST_LIBRARY AND BROADCOM_INCLUDE_DIR)

  PKG_CHECK_MODULES(VDPAU vdpau)
  IF(VDPAU_FOUND)
    IF(VA_DRM_LIBRARY)
      SET(HAVE_VDPAU_DRM 1)
    ENDIF(VA_DRM_LIBRARY)
    IF(X11_FOUND)
      SET(HAVE_VDPAU_X11 1)
    ENDIF(X11_FOUND)
    ADD_DEFINITIONS(-DHAVE_VDPAU)
    SET(CONFIG_VDPAU 1)
    SET(DEPENDENS_INCLUDE_DIRS ${DEPENDENS_INCLUDE_DIRS} ${VDPAU_INCLUDE_DIRS})
    SET(DEPENDENS_LIBRARIES ${DEPENDENS_LIBRARIES} ${VDPAU_LIBRARIES})
  ENDIF(VDPAU_FOUND)

  PKG_CHECK_MODULES(VAAPI libva)
  IF(VAAPI_FOUND)
    IF(VA_DRM_LIBRARY)
      SET(HAVE_VAAPI_DRM 1)
    ENDIF(VA_DRM_LIBRARY)
    IF(X11_FOUND)
      SET(HAVE_VAAPI_X11 1)
    ENDIF(X11_FOUND)
    ADD_DEFINITIONS(-DHAVE_VAAPI)
    SET(HAVE_VAAPI 1)
    SET(CONFIG_VAAPI 1)
    SET(DEPENDENS_INCLUDE_DIRS ${DEPENDENS_INCLUDE_DIRS} ${VAAPI_INCLUDE_DIRS})
    SET(DEPENDENS_LIBRARIES ${DEPENDENS_LIBRARIES} ${VAAPI_LIBRARIES})
  ENDIF(VAAPI_FOUND)

  FIND_LIBRARY(ATOMIC_LIBRARY NAMES atomic atomic.so.1 libatomic.so.1)
  IF(ATOMIC_LIBRARY)
    SET(PLATFORM_LIBRARIES ${PLATFORM_LIBRARIES} ${ATOMIC_LIBRARY})
  ENDIF(ATOMIC_LIBRARY)

  SET(PLATFORM_HDRS ${PLATFORM_HDRS})
  SET(PLATFORM_SRCS ${PLATFORM_SRCS})
  SET(PLATFORM_LIBRARIES ${PLATFORM_LIBRARIES} dl m)
ELSEIF(OS_FREEBSD)
  SET(PLATFORM_HDRS ${PLATFORM_HDRS})
  SET(PLATFORM_SRCS ${PLATFORM_SRCS})
  SET(PLATFORM_LIBRARIES ${PLATFORM_LIBRARIES})
ENDIF(OS_WINDOWS)

FIND_PACKAGE(FFmpeg REQUIRED)
FIND_PACKAGE(Common REQUIRED)
FIND_PACKAGE(SDL2 REQUIRED)
FIND_PACKAGE(Freetype REQUIRED)
FIND_PACKAGE(OpenSSL REQUIRED)

IF(FFMPEG_LIBAVFILTER)
  SET(CONFIG_AVFILTER 1)
ENDIF(FFMPEG_LIBAVFILTER)
IF (NOT CONFIG_AVFILTER)
  MESSAGE(FATAL_ERROR "REQUEIRED AVFILER SUPPORT!!!")
ENDIF(NOT CONFIG_AVFILTER)
IF(FFMPEG_LIBAVDEVICE)
  SET(CONFIG_AVDEVICE 1)
ENDIF(FFMPEG_LIBAVDEVICE)
IF(FFMPEG_LIBAVUTIL)
  SET(CONFIG_AVUTIL 1)
ENDIF(FFMPEG_LIBAVUTIL)
IF(FFMPEG_LIBAVCODEC)
  SET(CONFIG_AVCODEC 1)
ENDIF(FFMPEG_LIBAVCODEC)
IF(FFMPEG_LIBAVFORMAT)
  SET(CONFIG_AVFORMAT 1)
ENDIF(FFMPEG_LIBAVFORMAT)
IF(FFMPEG_LIBSWSCALE)
  SET(CONFIG_SWSCALE 1)
ENDIF(FFMPEG_LIBSWSCALE)
IF(FFMPEG_LIBSWRESAMPLE)
  SET(CONFIG_SWRESAMPLE 1)
ENDIF(FFMPEG_LIBSWRESAMPLE)

FIND_PACKAGE(FastoTvCPP REQUIRED)
FIND_PACKAGE(FastoPlayer REQUIRED)
SET_DESKTOP_TARGET()

# Config
SET(CONFIG_HWACCEL_METHOD "none" CACHE STRING "Hwaccel method")
SET(CONFIG_WIDTH -1 CACHE STRING "Player width")
SET(CONFIG_HEIGHT -1 CACHE STRING "Player height")
SET(CONFIG_POWER_OFF_ON_EXIT OFF CACHE BOOL "Power off device on exit")

SET(CONFIG_FILE_NAME ${PROJECT_NAME_LOWERCASE}.ini)
IF(OS_WINDOWS)
  SET(EXECUTABLE_FOLDER_PATH ${TARGET_INSTALL_DESTINATION})
ELSEIF(OS_MACOSX)
  SET(EXECUTABLE_FOLDER_PATH ${TARGET_INSTALL_DESTINATION})
ELSEIF(OS_LINUX OR OS_FREEBSD)
  SET(EXECUTABLE_FOLDER_PATH /usr/bin)
ENDIF(OS_WINDOWS)

#config file
SET(CONFIG_FILE_GENERATED_PATH "${CMAKE_CURRENT_BINARY_DIR}/${CONFIG_FILE_NAME}")
CONFIGURE_FILE("${PROJECT_BRANDING_FOLDER}/config.ini.in"
  ${CONFIG_FILE_GENERATED_PATH} @ONLY IMMEDIATE)
SET(CONFIG_FILE_PATH_RELATIVE ${SHARE_INSTALL_DESTINATION}/${CONFIG_FILE_NAME})

SET(EXECUTABLE_PATH ${EXECUTABLE_FOLDER_PATH}/${PROJECT_NAME_LOWERCASE} CACHE INTERNAL
  "Executable path: ${EXECUTABLE_PATH}") # used for (deb/rpm/xinitrc)

SET(APPLICATION_DIR "~/.${PROJECT_NAME_LOWERCASE}")
SET(LOG_FILE_NAME "${PROJECT_NAME_LOWERCASE}.log")

ADD_DEFINITIONS(
  -DLOG_FILE_NAME="${LOG_FILE_NAME}"
  -DAPPLICATION_DIR="${APPLICATION_DIR}"

  -DCONFIG_FILE_NAME="${CONFIG_FILE_NAME}"
  -DCONFIG_FILE_PATH_RELATIVE="${CONFIG_FILE_PATH_RELATIVE}"
)

# simple player
SET(BUILD_PLAYER_SOURCES
  ${CLIENT_SOURCE_DIR}/load_config.h
  ${CLIENT_SOURCE_DIR}/load_config.cpp
  ${CLIENT_SOURCE_DIR}/cmdutils.h
  ${CLIENT_SOURCE_DIR}/cmdutils.cpp
)

ADD_SUBDIRECTORY(${SOURCE_ROOT}/third-party/ini)

#tv player
FIND_PACKAGE(PNG REQUIRED)
FIND_PACKAGE(JSON-C REQUIRED)
FIND_PACKAGE(Cpuid REQUIRED)
FIND_PACKAGE(LircClient QUIET)
FIND_PACKAGE(LibEv REQUIRED)
IF(LIRC_CLIENT_FOUND)
  ADD_DEFINITIONS(-DHAVE_LIRC)
  SET(DEPENDENS_CLIENT_SOURCES ${DEPENDENS_CLIENT_SOURCES} ${CLIENT_SOURCE_DIR}/inputs/lirc_input_client.h)
  SET(DEPENDENS_CLIENT_HEADERS ${DEPENDENS_CLIENT_HEADERS} ${CLIENT_SOURCE_DIR}/inputs/lirc_input_client.cpp)
  SET(DEPENDENS_CLIENT_INCLUDE_DIRS ${DEPENDENS_CLIENT_INCLUDE_DIRS} ${LIRC_CLIENT_INCLUDE_DIR})
  SET(DEPENDENS_CLIENT_LIBRARIES ${DEPENDENS_CLIENT_LIBRARIES} ${LIRC_CLIENT_LIBRARIES})
ENDIF(LIRC_CLIENT_FOUND)

SET(HEADERS_EVENTS_CLIENT
  ${CLIENT_SOURCE_DIR}/events/network_events.h
)

SET(SOURCES_EVENTS_CLIENT
  ${CLIENT_SOURCE_DIR}/events/network_events.cpp
)

SET(HEADERS_INNER_CLIENT
  ${CLIENT_SOURCE_DIR}/inner/inner_tcp_server.h
  ${CLIENT_SOURCE_DIR}/inner/inner_tcp_handler.h
)

SET(SOURCES_INNER_CLIENT
  ${CLIENT_SOURCE_DIR}/inner/inner_tcp_server.cpp
  ${CLIENT_SOURCE_DIR}/inner/inner_tcp_handler.cpp
)

SET(LIVE_STREAM_SOURCES
  ${CLIENT_SOURCE_DIR}/live_stream/playlist_entry.h
  ${CLIENT_SOURCE_DIR}/live_stream/playlist_entry.cpp
  ${CLIENT_SOURCE_DIR}/live_stream/playlist_window.h
  ${CLIENT_SOURCE_DIR}/live_stream/playlist_window.cpp
)

SET(VOD_STREAM_SOURCES
  ${CLIENT_SOURCE_DIR}/vod/vod_entry.h
  ${CLIENT_SOURCE_DIR}/vod/vod_entry.cpp
  ${CLIENT_SOURCE_DIR}/vod/vods_window.h
  ${CLIENT_SOURCE_DIR}/vod/vods_window.cpp
)

SET(TV_PLAYER_SOURCES
  ${CLIENT_SOURCE_DIR}/ioservice.h
  ${CLIENT_SOURCE_DIR}/ioservice.cpp
  ${CLIENT_SOURCE_DIR}/utils.h
  ${CLIENT_SOURCE_DIR}/utils.cpp

  ${CLIENT_SOURCE_DIR}/player.h
  ${CLIENT_SOURCE_DIR}/player.cpp
  ${CLIENT_SOURCE_DIR}/programs_window.h
  ${CLIENT_SOURCE_DIR}/programs_window.cpp
  ${CLIENT_SOURCE_DIR}/vods_window.h
  ${CLIENT_SOURCE_DIR}/vods_window.cpp

  ${LIVE_STREAM_SOURCES}
  ${VOD_STREAM_SOURCES}
  ${BUILD_PLAYER_SOURCES}
  ${HEADERS_INNER_CLIENT} ${SOURCES_INNER_CLIENT}
  ${HEADERS_EVENTS_CLIENT} ${SOURCES_EVENTS_CLIENT}
  ${DEPENDENS_CLIENT_SOURCES} ${DEPENDENS_CLIENT_HEADERS}
)
SET(EXE_TV_PLAYER_SOURCES
  ${CLIENT_SOURCE_DIR}/tv_player_main.cpp
  ${TV_PLAYER_SOURCES}
)
SET(OTHER_TV_PLAYER_SOURCES ${ICON_FILE} ${RESOURCE_OS})

SET(TV_PLAYER_INCLUDE_DIRECTORIES
  ${SOURCE_ROOT}
  ${SOURCE_ROOT}/third-party/ini
  ${DEPENDENS_CLIENT_INCLUDE_DIRS}
  ${COMMON_INCLUDE_DIRS}
  ${FASTOTV_CPP_INCLUDE_DIRS}
  ${FASTO_PLAYER_INCLUDE_DIRS}
  ${SDL2_INCLUDE_DIRS}
  ${FREETYPE_INCLUDE_DIRS}
  ${DEPENDENS_CLIENT_INCLUDE_DIRS}
  ${LIBEV_INCLUDE_DIRS}
  ${JSONC_INCLUDE_DIRS}
  ${CPUID_INCLUDE_DIRS}
)

SET(TV_PLAYER_LIBRARIES
  ${FASTO_PLAYER_LIBRARIES}
  ${FASTOTV_CPP_LIBRARIES}
  ${FFMPEG_LIBRARIES}
  ${COMMON_EV_LIBRARIES}
  ${COMMON_BASE_LIBRARY}
  ${JSONC_LIBRARIES}
  inih
  ${DEPENDENS_CLIENT_LIBRARIES}
  ${PNG_LIBRARIES}
  ${BZIP2_LIBRARIES}
  ${ZLIB_LIBRARIES}
  ${OPENSSL_LIBRARIES}
  ${SDL2_LIBRARIES}
  ${FREETYPE_LIBRARIES}
  ${LIBEV_LIBRARIES}
  ${CPUID_LIBRARIES}
  ${PLATFORM_LIBRARIES}
  ${DEPENDENS_LIBRARIES}
)

IF(MINGW OR CMAKE_COMPILER_IS_GNUCXX OR CMAKE_COMPILER_IS_CLANGCXX)
  ADD_EXECUTABLE(${PROJECT_NAME} ${DESKTOP_TARGET} ${EXE_TV_PLAYER_SOURCES} ${OTHER_TV_PLAYER_SOURCES})
  TARGET_INCLUDE_DIRECTORIES(${PROJECT_NAME} PRIVATE ${TV_PLAYER_INCLUDE_DIRECTORIES})
  TARGET_LINK_LIBRARIES(${PROJECT_NAME} ${TV_PLAYER_LIBRARIES})
ELSE()
  MESSAGE(FATAL_ERROR "NOT SUPPORTED COMPILER!!!")
ENDIF(MINGW OR CMAKE_COMPILER_IS_GNUCXX OR CMAKE_COMPILER_IS_CLANGCXX)

IF(OS_WINDOWS)
  SET_SOURCE_FILES_PROPERTIES(${RESOURCE_OS} PROPERTIES LANGUAGE RC)
ELSEIF(OS_MACOSX)
  SET_SOURCE_FILES_PROPERTIES(${RESOURCE_OS} PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
  SET_SOURCE_FILES_PROPERTIES(${ICON_FILE} PROPERTIES MACOSX_PACKAGE_LOCATION Resources)
ENDIF(OS_WINDOWS)

GET_FILENAME_COMPONENT(ICON_FILE_NAME ${ICON_FILE} NAME)
IF(OS_MACOSX)
  SET(MACOSX_BUNDLE_BUNDLE_NAME ${BUNDLE_BASE_NAME})
  SET(MACOSX_BUNDLE_GUI_IDENTIFIER ${PROJECT_NAME})
  SET(MACOSX_BUNDLE_INFO_STRING "${PROJECT_VERSION},${PROJECT_COPYRIGHT}" )
  SET(MACOSX_BUNDLE_SHORT_VERSION_STRING "${SHORT_VERSION}" )
  SET(MACOSX_BUNDLE_LONG_VERSION_STRING ${PROJECT_VERSION})
  SET(MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION})
  SET(MACOSX_BUNDLE_COPYRIGHT ${PROJECT_COPYRIGHT})
  SET(MACOSX_BUNDLE_ICON_FILE ${ICON_FILE_NAME})
  SET(MACOSX_BUNDLE_PRINCIPAL_CLASS "NSApplication")
  #SET(MACOSX_MAIN_NIB_FILE "MainMenu")
ENDIF(OS_MACOSX)

#prepare executable
IF(PROJECT_BUILD_TYPE_VERSION STREQUAL "release")
  STRIP_TARGET(${PROJECT_NAME})
  SIGN_TARGET(${EXECUTABLE_NAME})
ENDIF(PROJECT_BUILD_TYPE_VERSION STREQUAL "release")

# Start to install
VersionConf(${PROJECT_NAME} ${RESOURCE_OS_IN} ${RESOURCE_OS} ${ICON_FILE_NAME})
INSTALL(TARGETS ${PROJECT_NAME} DESTINATION ${TARGET_INSTALL_DESTINATION} COMPONENT APPLICATIONS)
INSTALL(FILES "${CMAKE_SOURCE_DIR}/LICENSE" DESTINATION . COMPONENT LICENSE RENAME LICENSE)
INSTALL(FILES "${PROJECT_BRANDING_FOLDER}/COPYRIGHT" DESTINATION . COMPONENT LICENSE RENAME COPYRIGHT)
INSTALL(FILES ${PROJECT_CHANGELOG_FILE} DESTINATION . COMPONENT LICENSE RENAME CHANGELOG)

#share install
IF(LIRC_CLIENT_FOUND)
  SET(LIRCRC_CONFIG_NAME "lircrc.conf")
  SET(LIRCRC_CONFIG_GENERATED_PATH "${PROJECT_CURRENT_BINARY_DIR}/${LIRCRC_CONFIG_NAME}")
  SET(LIRCRC_CONFIG_PATH_RELATIVE ${SHARE_INSTALL_DESTINATION}/${LIRCRC_CONFIG_NAME})
  ADD_DEFINITIONS(-DLIRCRC_CONFIG_PATH_RELATIVE="${LIRCRC_CONFIG_PATH_RELATIVE}")
  CONFIGURE_FILE("${PROJECT_BRANDING_FOLDER}/hardware/lirc/${LIRCRC_CONFIG_NAME}.in"
    ${LIRCRC_CONFIG_GENERATED_PATH} @ONLY IMMEDIATE)
  INSTALL(FILES ${LIRCRC_CONFIG_GENERATED_PATH} DESTINATION
    ${SHARE_INSTALL_DESTINATION} COMPONENT RESOURCES)
ENDIF(LIRC_CLIENT_FOUND)

INSTALL(FILES ${CONFIG_FILE_GENERATED_PATH} DESTINATION ${SHARE_INSTALL_DESTINATION} COMPONENT RUNTIME)
INSTALL(DIRECTORY ${PROJECT_BRANDING_FOLDER}/resources DESTINATION
  ${SHARE_INSTALL_DESTINATION} COMPONENT RESOURCES)
INSTALL(DIRECTORY ${PROJECT_BRANDING_FOLDER}/fonts DESTINATION
  ${SHARE_INSTALL_DESTINATION} COMPONENT RESOURCES)

IF(OS_WINDOWS)
  #find runtime zlib
  SET(SHARED_ZLIB_NAMES zlib1.dll z.dll zlib.dll zdll.dll zlibd.dll zlibd1.dll)
  FIND_RUNTIME_LIBRARY(SHARED_ZLIB_LIBRARY SHARED_ZLIB_NAMES)
  INSTALL(FILES ${SHARED_ZLIB_LIBRARY} DESTINATION ${LIB_INSTALL_DESTINATION} COMPONENT RUNTIME)

  #find runtime bz
  SET(SHARED_BZ2_NAMES libbz2-1.dll)
  FIND_RUNTIME_LIBRARY(SHARED_BZ2_LIBRARY SHARED_BZ2_NAMES)
  INSTALL(FILES ${SHARED_BZ2_LIBRARY} DESTINATION ${LIB_INSTALL_DESTINATION} COMPONENT RUNTIME)

  #find runtime png
  SET(SHARED_PNG_NAMES libpng16-16.dll)
  FIND_RUNTIME_LIBRARY(SHARED_PNG_LIBRARY SHARED_PNG_NAMES)
  INSTALL(FILES ${SHARED_PNG_LIBRARY} DESTINATION ${LIB_INSTALL_DESTINATION} COMPONENT RUNTIME)

  #find runtime harfbuzz
  SET(SHARED_HARFBUZZ_NAMES libharfbuzz-0.dll)
  FIND_RUNTIME_LIBRARY(SHARED_HARFBUZZ_LIBRARY SHARED_HARFBUZZ_NAMES)
  INSTALL(FILES ${SHARED_HARFBUZZ_LIBRARY} DESTINATION ${LIB_INSTALL_DESTINATION} COMPONENT RUNTIME)

  #find runtime glib
  SET(SHARED_GLIB_NAMES libglib-2.0-0.dll)
  FIND_RUNTIME_LIBRARY(SHARED_GLIB_LIBRARY SHARED_GLIB_NAMES)
  INSTALL(FILES ${SHARED_GLIB_LIBRARY} DESTINATION ${LIB_INSTALL_DESTINATION} COMPONENT RUNTIME)

  #find runtime intl
  SET(SHARED_INTL_NAMES libintl-8.dll)
  FIND_RUNTIME_LIBRARY(SHARED_INTL_LIBRARY SHARED_INTL_NAMES)
  INSTALL(FILES ${SHARED_INTL_LIBRARY} DESTINATION ${LIB_INSTALL_DESTINATION} COMPONENT RUNTIME)

  #find runtime pcre
  SET(SHARED_PCRE_NAMES libpcre-1.dll)
  FIND_RUNTIME_LIBRARY(SHARED_PCRE_LIBRARY SHARED_PCRE_NAMES)
  INSTALL(FILES ${SHARED_PCRE_LIBRARY} DESTINATION ${LIB_INSTALL_DESTINATION} COMPONENT RUNTIME)

  #find runtime iconv
  SET(SHARED_ICONV_NAMES libiconv-2.dll)
  FIND_RUNTIME_LIBRARY(SHARED_ICONV_LIBRARY SHARED_ICONV_NAMES)
  INSTALL(FILES ${SHARED_ICONV_LIBRARY} DESTINATION ${LIB_INSTALL_DESTINATION} COMPONENT RUNTIME)

  #find runtime graphite2
  SET(SHARED_GRAPHITE2_NAMES libgraphite2.dll)
  FIND_RUNTIME_LIBRARY(SHARED_GRAPHITE2_LIBRARY SHARED_GRAPHITE2_NAMES)
  INSTALL(FILES ${SHARED_GRAPHITE2_LIBRARY} DESTINATION ${LIB_INSTALL_DESTINATION} COMPONENT RUNTIME)

  #find runtime freetype
  SET(SHARED_FREETYPE_NAMES libfreetype-6.dll)
  FIND_RUNTIME_LIBRARY(SHARED_FREETYPE_LIBRARY SHARED_FREETYPE_NAMES)
  INSTALL(FILES ${SHARED_FREETYPE_LIBRARY} DESTINATION ${LIB_INSTALL_DESTINATION} COMPONENT RUNTIME)
  
  #find runtime sdl2
  SET(SHARED_SDL2_NAMES SDL2.dll)
  FIND_RUNTIME_LIBRARY(SHARED_SDL2_LIBRARY SHARED_SDL2_NAMES)
  INSTALL(FILES ${SHARED_SDL2_LIBRARY} DESTINATION ${LIB_INSTALL_DESTINATION} COMPONENT RUNTIME)

  #find runtime sdl2 image
  SET(SHARED_SDL2_IMAGE_NAMES SDL2_image.dll)
  FIND_RUNTIME_LIBRARY(SHARED_SDL2_IMAGE_LIBRARY SHARED_SDL2_IMAGE_NAMES)
  INSTALL(FILES ${SHARED_SDL2_IMAGE_LIBRARY} DESTINATION ${LIB_INSTALL_DESTINATION} COMPONENT RUNTIME)

  #find runtime sdl2 ttf
  SET(SHARED_SDL2_TTF_NAMES SDL2_ttf.dll)
  FIND_RUNTIME_LIBRARY(SHARED_SDL2_TTF_LIBRARY SHARED_SDL2_TTF_NAMES)
  INSTALL(FILES ${SHARED_SDL2_TTF_LIBRARY} DESTINATION ${LIB_INSTALL_DESTINATION} COMPONENT RUNTIME)
ELSEIF(OS_MACOSX)
  SET(BUNDLE_PATH "${CMAKE_INSTALL_PREFIX}/${BUNDLE_NAME}")
  INSTALL_LIB_TO_BUNDLE(${BUNDLE_NAME} ${SDL2_LIBRARY})
  INSTALL_LIB_TO_BUNDLE(${BUNDLE_NAME} ${SDL2_IMAGE_LIBRARY})
  INSTALL_LIB_TO_BUNDLE(${BUNDLE_NAME} ${SDL2_TTF_LIBRARY})
ELSEIF(OS_LINUX OR OS_FREEBSD)
  GET_FILENAME_COMPONENT(LIBPNG_LIBRARY_WITHOUT_SYMLINK ${PNG_LIBRARY} REALPATH)
  GET_FILENAME_COMPONENT(LIBPNG_LIBRARY_NAME ${LIBPNG_LIBRARY_WITHOUT_SYMLINK} NAME)
  STRING(REGEX REPLACE "[^so]+$" ".0" LIBPNG_LNNAME ${LIBPNG_LIBRARY_NAME})
  #libpng12.so.0
  INSTALL(FILES ${LIBPNG_LIBRARY_WITHOUT_SYMLINK} DESTINATION ${LIB_INSTALL_DESTINATION} RENAME ${LIBPNG_LNNAME} COMPONENT RUNTIME)

  GET_FILENAME_COMPONENT(SDL2_LIBRARY_WITHOUT_SYMLINK ${SDL2_LIBRARY} REALPATH)
  GET_FILENAME_COMPONENT(SDL2_LIBRARY_NAME ${SDL2_LIBRARY_WITHOUT_SYMLINK} NAME)
  STRING(REGEX REPLACE "[^so]+$" ".0" SDL2_LNNAME ${SDL2_LIBRARY_NAME})
  #libSDL2-2.0.so.0
  INSTALL(FILES ${SDL2_LIBRARY_WITHOUT_SYMLINK} DESTINATION ${LIB_INSTALL_DESTINATION} RENAME ${SDL2_LNNAME} COMPONENT RUNTIME)

  GET_FILENAME_COMPONENT(SDL2_IMAGE_LIBRARY_WITHOUT_SYMLINK ${SDL2_IMAGE_LIBRARY} REALPATH)
  GET_FILENAME_COMPONENT(SDL2_IMAGE_LIBRARY_NAME ${SDL2_IMAGE_LIBRARY_WITHOUT_SYMLINK} NAME)
  STRING(REGEX REPLACE "[^so]+$" ".0" SDL2_IMAGE_LNNAME ${SDL2_IMAGE_LIBRARY_NAME})
  #libSDL2_image-2.0.so.0
  INSTALL(FILES ${SDL2_IMAGE_LIBRARY_WITHOUT_SYMLINK} DESTINATION ${LIB_INSTALL_DESTINATION} RENAME ${SDL2_IMAGE_LNNAME} COMPONENT RUNTIME)

  GET_FILENAME_COMPONENT(SDL2_TTF_LIBRARY_WITHOUT_SYMLINK ${SDL2_TTF_LIBRARY} REALPATH)
  GET_FILENAME_COMPONENT(SDL2_TTF_LIBRARY_NAME ${SDL2_TTF_LIBRARY_WITHOUT_SYMLINK} NAME)
  STRING(REGEX REPLACE "[^so]+$" ".0" SDL2_TTF_LNNAME ${SDL2_TTF_LIBRARY_NAME})
  #libSDL2_ttf-2.0.so.0
  INSTALL(FILES ${SDL2_TTF_LIBRARY_WITHOUT_SYMLINK} DESTINATION ${LIB_INSTALL_DESTINATION} RENAME ${SDL2_TTF_LNNAME} COMPONENT RUNTIME)

  IF(LIRC_CLIENT_FOUND)
    GET_FILENAME_COMPONENT(LIRC_LIBRARY_WITHOUT_SYMLINK ${LIRC_CLIENT_LIBRARY} REALPATH)
    GET_FILENAME_COMPONENT(LIRC_LIBRARY_NAME ${LIRC_LIBRARY_WITHOUT_SYMLINK} NAME)
    STRING(REGEX REPLACE "[^so]+$" ".0" LIRC_LNNAME ${LIRC_LIBRARY_NAME})
    #liblirc_client.so.0
    INSTALL(FILES ${LIRC_LIBRARY_WITHOUT_SYMLINK} DESTINATION ${LIB_INSTALL_DESTINATION} RENAME ${LIRC_LNNAME} COMPONENT RUNTIME)
  ENDIF(LIRC_CLIENT_FOUND)

  # va drm
  IF(VA_DRM_LIBRARY)
    GET_FILENAME_COMPONENT(VA_DRM_LIBRARY_WITHOUT_SYMLINK ${VA_DRM_LIBRARY} REALPATH)
    GET_FILENAME_COMPONENT(VA_DRM_LIBRARY_NAME ${VA_DRM_LIBRARY_WITHOUT_SYMLINK} NAME)
    STRING(REGEX REPLACE "[^so]+$" ".1" VA_DRM_LNNAME ${VA_DRM_LIBRARY_NAME})
    #libva-drm.so.1
    INSTALL(FILES ${VA_DRM_LIBRARY_WITHOUT_SYMLINK} DESTINATION ${LIB_INSTALL_DESTINATION} RENAME ${VA_DRM_LNNAME} COMPONENT RUNTIME)
  ENDIF(VA_DRM_LIBRARY)

  # va x11
  IF(VA_X11_LIBRARY)
    GET_FILENAME_COMPONENT(VA_X11_LIBRARY_WITHOUT_SYMLINK ${VA_X11_LIBRARY} REALPATH)
    GET_FILENAME_COMPONENT(VA_X11_LIBRARY_NAME ${VA_X11_LIBRARY_WITHOUT_SYMLINK} NAME)
    STRING(REGEX REPLACE "[^so]+$" ".1" VA_X11_LNNAME ${VA_X11_LIBRARY_NAME})
    #libva-x11.so.1
    INSTALL(FILES ${VA_X11_LIBRARY_WITHOUT_SYMLINK} DESTINATION ${LIB_INSTALL_DESTINATION} RENAME ${VA_X11_LNNAME} COMPONENT RUNTIME)
  ENDIF(VA_X11_LIBRARY)

  # vdpau
  IF(VDPAU_FOUND)
    FIND_LIBRARY(VDPAU_LIBRARY NAMES vdpau DOC "Path to vdpau library")
    GET_FILENAME_COMPONENT(VDPAU_LIBRARY_WITHOUT_SYMLINK ${VDPAU_LIBRARY} REALPATH)
    GET_FILENAME_COMPONENT(VDPAU_LIBRARY_NAME ${VDPAU_LIBRARY_WITHOUT_SYMLINK} NAME)
    STRING(REGEX REPLACE "[^so]+$" ".1" VDPAU_LNNAME ${VDPAU_LIBRARY_NAME})
    #libvdpau.so.1
    INSTALL(FILES ${VDPAU_LIBRARY_WITHOUT_SYMLINK} DESTINATION ${LIB_INSTALL_DESTINATION} RENAME ${VDPAU_LNNAME} COMPONENT RUNTIME)
  ENDIF(VDPAU_FOUND)

  # va
  IF(VAAPI_FOUND)
    FIND_LIBRARY(VA_LIBRARY NAMES va DOC "Path to va library")
    GET_FILENAME_COMPONENT(VA_LIBRARY_WITHOUT_SYMLINK ${VA_LIBRARY} REALPATH)
    GET_FILENAME_COMPONENT(VA_LIBRARY_NAME ${VA_LIBRARY_WITHOUT_SYMLINK} NAME)
    STRING(REGEX REPLACE "[^so]+$" ".1" VA_LNNAME ${VA_LIBRARY_NAME})
    #libva.so.1
    INSTALL(FILES ${VA_LIBRARY_WITHOUT_SYMLINK} DESTINATION ${LIB_INSTALL_DESTINATION} RENAME ${VA_LNNAME} COMPONENT RUNTIME)
  ENDIF(VAAPI_FOUND)

  # exe script
  SET(START_SCRIPT_TV_PLAYER_GENERATED_PATH "${PROJECT_CURRENT_BINARY_DIR}/${PROJECT_NAME_LOWERCASE}")
  GEN_START_SCRIPT(${START_SCRIPT_TV_PLAYER_GENERATED_PATH} ${PROJECT_NAME})
  INSTALL(PROGRAMS ${START_SCRIPT_TV_PLAYER_GENERATED_PATH} DESTINATION ${TARGET_INSTALL_DESTINATION})

  #share resources

  #xinitrc script
  SET(XINITRC_SCRIPT_GENERATED_PATH "${PROJECT_CURRENT_BINARY_DIR}/xinitrc")
  CONFIGURE_FILE("${PROJECT_BRANDING_FOLDER}/linux/xinitrc.in"
    ${XINITRC_SCRIPT_GENERATED_PATH} @ONLY IMMEDIATE)
  INSTALL(FILES ${XINITRC_SCRIPT_GENERATED_PATH} DESTINATION ${SHARE_INSTALL_DESTINATION} COMPONENT RESOURCES)

  # auto_start script
  SET(AURO_START_SCRIPT_GENERATED_PATH "${PROJECT_CURRENT_BINARY_DIR}/autostart_${PROJECT_NAME_LOWERCASE}")
  CONFIGURE_FILE("${PROJECT_BRANDING_FOLDER}/autostart.sh.in"
    ${AURO_START_SCRIPT_GENERATED_PATH} @ONLY IMMEDIATE)
  INSTALL(PROGRAMS ${AURO_START_SCRIPT_GENERATED_PATH} DESTINATION ${SHARE_INSTALL_DESTINATION})

  INSTALL(FILES ${RESOURCE_OS} DESTINATION ${SHARE_INSTALL_DESTINATION}/applications COMPONENT RESOURCES)
  INSTALL(FILES ${ICON_FILE} DESTINATION ${SHARE_INSTALL_DESTINATION}/icons COMPONENT RESOURCES)
ENDIF(OS_WINDOWS)

INSTALL_RUNTIME_LIBRARIES()

IF(DEVELOPER_CHECK_STYLE)
  SET(CHECK_SOURCES_CLIENT ${TV_PLAYER_SOURCES} ${EXE_TV_PLAYER_SOURCES})
  REGISTER_CHECK_STYLE_TARGET(check_style_client "${CHECK_SOURCES_CLIENT}")
  REGISTER_CHECK_INCLUDES_TARGET(${PROJECT_NAME})
ENDIF(DEVELOPER_CHECK_STYLE)

IF(DEVELOPER_ENABLE_TESTS)
  IF(DEVELOPER_ENABLE_UNIT_TESTS)
    SET(PRIVATE_INCLUDE_DIRECTORIES_CLIENT_TEST
      ${gtest_SOURCE_DIR}/include ${gtest_SOURCE_DIR} ${SOURCE_ROOT}
      ${COMMON_INCLUDE_DIRS}
      ${JSONC_INCLUDE_DIRS}
    )

    SET(PROJECT_UNIT_TEST_CLIENT unit_tests_client)
    ADD_EXECUTABLE(${PROJECT_UNIT_TEST_CLIENT}
      ${CMAKE_SOURCE_DIR}/tests/unit_tests/test_commands.cpp
      ${CLIENT_SOURCE_DIR}/commands.cpp
    )
    TARGET_INCLUDE_DIRECTORIES(${PROJECT_UNIT_TEST_CLIENT} PRIVATE ${PRIVATE_INCLUDE_DIRECTORIES_CLIENT_TEST})
    TARGET_LINK_LIBRARIES(${PROJECT_UNIT_TEST_CLIENT} gtest gtest_main
      ${PROJECT_CLIENT_SERVER_LIBRARY} ${COMMON_BASE_LIBRARY} ${JSONC_LIBRARIES} ${PLATFORM_LIBRARIES}
    )
    ADD_TEST_TARGET(${PROJECT_UNIT_TEST_CLIENT})
    SET_PROPERTY(TARGET ${PROJECT_UNIT_TEST_CLIENT} PROPERTY FOLDER "Unit tests")
  ENDIF(DEVELOPER_ENABLE_UNIT_TESTS)
ENDIF(DEVELOPER_ENABLE_TESTS)
