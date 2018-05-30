cmake_minimum_required(VERSION 3.0.2)

include(CryptoNoteWallet.cmake)

project(${CN_PROJECT_NAME})

if (WIN32)
  message(STATUS "${PROJECT_BINARY_DIR}")
  find_package(Git)
  find_package(Boost 1.58 REQUIRED COMPONENTS
      date_time
      filesystem
      program_options
      regex
      serialization
      system
      thread
      chrono)

  find_package(Qt5 REQUIRED COMPONENTS
      Core
      Gui
      Widgets
      Network
      DBus)

  #set(EXTERNAL_CMAKE_ARGS
  #    -DSTATIC:BOOL=OFF
  #    -DBoost_USE_STATIC_LIBS:BOOL=ON
  #    -DBoost_USE_STATIC_RUNTIME:BOOL=OFF)
  #include(ExternalProject)
  #ExternalProject_Add(BBSCoin
  #    SOURCE_DIR ${CMAKE_SOURCE_DIR}/bbscoin
  #    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/BBSCoin
  #    CMAKE_ARGS 
  #        -DSTATIC:BOOL=OFF
  #        -DBoost_USE_STATIC_LIBS:BOOL=ON
  #        -DBoost_USE_STATIC_RUNTIME:BOOL=OFF
  #    INSTALL_COMMAND ""
  #    )

  set(Boost_USE_STATIC_LIBS ON)
  set(Boost_USE_STATIC_RUNTIME OFF)

  # Get and add-definition for AboutDialog.cpp
  # Set something in case git is not found
  set(GIT_REVISION "unknown")
  if (GIT_FOUND)
      execute_process(
          COMMAND ${GIT_EXECUTABLE} rev-parse --short HEAD
          WORKING_DIRECTORY "${CMAKE_SOURCE_DIR}"
          OUTPUT_VARIABLE GIT_REVISION
          OUTPUT_STRIP_TRAILING_WHITESPACE
      )
      message(STATUS "Git Revision ${GIT_REVISION}")
  else()
      message(STATUS "Git not found")
  endif()

  configure_file("${PROJECT_SOURCE_DIR}/src/CryptoNoteWalletConfig.h.in" "${PROJECT_BINARY_DIR}/CryptoNoteWalletConfig.h")

  set(CMAKE_AUTOMOC ON)
  set(CMAKE_AUTOUIC ON)
  set(CMAKE_AUTORCC ON)
  set(CMAKE_INCLUDE_CURRENT_DIR ON)

  file(GLOB_RECURSE SOURCES src/*.cpp)
  file(GLOB_RECURSE HEADERS src/*.h)
  file(GLOB_RECURSE FORMS src/gui/ui/*.ui)

  set(SOURCES ${SOURCES} Platform/Windows/ApplicationEventHandler.cpp)

  set(QRC src/resources.qrc)

  qt5_wrap_ui(UIS ${FORMS})
  qt5_add_resources(RCC ${QRC})


  if(MSVC)
    add_definitions(/D_CRT_SECURE_NO_WARNINGS /D_WIN32_WINNT=0x0600)
    include_directories(
        bbscoin/src/Platform/msc
        bbscoin/src/Platform/Windows
        Platform/Windows)
    set(BUILD_PLATFORM WIN32)
    set(BUILD_RESOURCES src/bbscoinwallet.rc)
    set(Qt5_MAIN Qt5::WinMain)
  endif()

  # Include boost
  #include_directories(${Boost_INCLUDE_DIRS})
  # Link our externalproject *.lib's
  link_directories(
      ${Boost_LIBRARY_DIRS}
      ${PROJECT_SOURCE_DIR}/libs)
  # Build BBSCoinWallet.exe
  add_executable(${PROJECT_NAME} ${BUILD_PLATFORM} ${BUILD_RESOURCES} ${SOURCES} ${HEADERS} ${UIS} ${RCC})
  set_target_properties(${PROJECT_NAME} PROPERTIES COMPILE_DEFINITIONS _GNU_SOURCE)
  target_compile_definitions(${PROJECT_NAME} PUBLIC -DGIT_REVISION="${GIT_REVISION}")
  target_include_directories(${PROJECT_NAME} PUBLIC
      include
      src
      bbscoin/include
      bbscoin/src
      bbscoin/external/rocksdb/include
      ${Boost_INCLUDE_DIR})
  target_link_libraries(${PROJECT_NAME}
      ${Qt5_MAIN}
      ${Qt5DBus_LIBRARIES}
      BlockchainExplorer
      Common
      Crypto
      CryptoNoteCore
      Http
      InProcessNode
      Logging
      NodeRpcProxy
      P2p
      Rpc
      Serialization
      System
      Transfers
      Wallet
      rocksdblib
      miniupnpc
      Qt5::Widgets
      Qt5::Gui
      Qt5::Network
      Imm32 Iphlpapi Rpcrt4 Winmm Wtsapi32) #WIN32 lib's
else()
  execute_process(COMMAND git log -1 --pretty=format:%h
  WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
  OUTPUT_VARIABLE GIT_REVISION)

  set(CRYPTONOTE_LIB cryptonote)
  set(STATIC ON)

  include_directories(${CMAKE_BINARY_DIR}
    src
    include
    cryptonote/external
    cryptonote/external/rocksdb/include/
    cryptonote/include
    cryptonote/src
  )

  find_package(Qt5Gui REQUIRED)
  find_package(Qt5Widgets REQUIRED)
  find_package(Qt5Network REQUIRED)
  find_package(Qt5DBus REQUIRED)

  include_directories(${Qt5DBus_INCLUDE_DIRS} cryptonote/external/rocksdb/include/ include src external "${CMAKE_CURRENT_BINARY_DIR}/version")

  if(STATIC)
    set(Boost_USE_STATIC_LIBS ON)
    set(Boost_USE_STATIC_RUNTIME ON)
  endif()

  find_package(Boost 1.58 REQUIRED COMPONENTS date_time filesystem program_options regex serialization system thread chrono)
  if ((${Boost_MAJOR_VERSION} EQUAL 1) AND (${Boost_MINOR_VERSION} EQUAL 54))
    message(SEND_ERROR "Boost version 1.58 is unsupported, more details are available here http://goo.gl/RrCFmA")
  endif ()

  include_directories(${Boost_INCLUDE_DIRS})
  link_directories(${Boost_LIBRARY_DIRS})

  set(VERSION "")
  configure_file("cryptonote/src/version.h.in" "version.h")
  configure_file("src/CryptoNoteWalletConfig.h.in" "CryptoNoteWalletConfig.h")

  add_definitions(-DGIT_REVISION="${GIT_REVISION}")

  set(CMAKE_AUTOMOC ON)

  file(GLOB_RECURSE CRYPTONOTE_SOURCES
    cryptonote/external/miniupnpc/connecthostport.c
    cryptonote/external/miniupnpc/igd_desc_parse.c
    cryptonote/external/miniupnpc/minisoap.c
    cryptonote/external/miniupnpc/miniupnpc.c
    cryptonote/external/miniupnpc/miniwget.c
    cryptonote/external/miniupnpc/minixml.c
    cryptonote/external/miniupnpc/portlistingparse.c
    cryptonote/external/miniupnpc/receivedata.c
    cryptonote/external/miniupnpc/upnpcommands.c
    cryptonote/external/miniupnpc/upnpreplyparse.c
    cryptonote/src/BlockchainExplorer/BlockchainExplorer.cpp
    cryptonote/src/BlockchainExplorer/BlockchainExplorerErrors.cpp
    cryptonote/src/Common/Base58.cpp
    cryptonote/src/Common/CommandLine.cpp
    cryptonote/src/Common/Util.cpp
    cryptonote/src/Common/StringTools.cpp
    cryptonote/src/Common/JsonValue.cpp
    cryptonote/src/Common/ConsoleTools.cpp
    cryptonote/src/Common/MemoryInputStream.cpp
    cryptonote/src/Common/PathTools.cpp
    cryptonote/src/Common/StdInputStream.cpp
    cryptonote/src/Common/StdOutputStream.cpp
    cryptonote/src/Common/StreamTools.cpp
    cryptonote/src/Common/StringOutputStream.cpp
    cryptonote/src/Common/StringView.cpp
    cryptonote/src/Common/VectorOutputStream.cpp
    cryptonote/src/Common/ScopeExit.cpp
    cryptonote/src/ConnectivityTool/ConnectivityTool.cpp
    cryptonote/src/crypto/blake256.c
    cryptonote/src/crypto/chacha8.c
    cryptonote/src/crypto/crypto-ops-data.c
    cryptonote/src/crypto/crypto-ops.c
    cryptonote/src/crypto/crypto.cpp
    cryptonote/src/crypto/groestl.c
    cryptonote/src/crypto/hash-extra-blake.c
    cryptonote/src/crypto/hash-extra-groestl.c
    cryptonote/src/crypto/hash-extra-jh.c
    cryptonote/src/crypto/hash-extra-skein.c
    cryptonote/src/crypto/hash.c
    cryptonote/src/crypto/jh.c
    cryptonote/src/crypto/keccak.c
    cryptonote/src/crypto/oaes_lib.c
    cryptonote/src/crypto/random.c
    cryptonote/src/crypto/skein.c
    cryptonote/src/crypto/slow-hash.c
    cryptonote/src/crypto/slow-hash.cpp
    cryptonote/src/crypto/tree-hash.c
    cryptonote/src/crypto/cn_context.cpp
    cryptonote/src/CryptoNoteCore/*.cpp
    cryptonote/src/CryptoNoteProtocol/*.cpp
    cryptonote/src/Daemon/*.cpp
    cryptonote/src/HTTP/*.cpp
    cryptonote/src/InProcessNode/InProcessNode.cpp
    cryptonote/src/InProcessNode/InProcessNodeErrors.cpp
    cryptonote/src/Logging/*.cpp
    cryptonote/src/NodeRpcProxy/NodeErrors.cpp
    cryptonote/src/NodeRpcProxy/NodeRpcProxy.cpp
    cryptonote/src/P2p/*.cpp
    cryptonote/src/Rpc/*.cpp
    cryptonote/src/Serialization/*.cpp
    cryptonote/src/Transfers/BlockchainSynchronizer.cpp
    cryptonote/src/Transfers/SynchronizationState.cpp
    cryptonote/src/Transfers/TransfersConsumer.cpp
    cryptonote/src/Transfers/TransfersContainer.cpp
    cryptonote/src/Transfers/TransfersSubscription.cpp
    cryptonote/src/Transfers/TransfersSynchronizer.cpp
    cryptonote/src/Wallet/*.cpp
    cryptonote/src/WalletLegacy/KeysStorage.cpp
    cryptonote/src/WalletLegacy/WalletLegacy.cpp
    cryptonote/src/WalletLegacy/WalletHelper.cpp
    cryptonote/src/WalletLegacy/WalletLegacySerializer.cpp
    cryptonote/src/WalletLegacy/WalletLegacySerialization.cpp
    cryptonote/src/WalletLegacy/WalletTransactionSender.cpp
    cryptonote/src/WalletLegacy/WalletUnconfirmedTransactions.cpp
    cryptonote/src/WalletLegacy/WalletUserTransactionsCache.cpp
    cryptonote/src/System/ContextGroup.cpp
    cryptonote/src/System/ContextGroupTimeout.cpp
    cryptonote/src/System/Event.cpp
    cryptonote/src/System/EventLock.cpp
    cryptonote/src/System/InterruptedException.cpp
    cryptonote/src/System/Ipv4Address.cpp
    cryptonote/src/System/TcpStream.cpp
    cryptonote/src/JsonRpcServer/*.cpp
    cryptonote/src/PaymentGate/*.cpp
    cryptonote/src/PaymentGateService/*.cpp
    cryptonote/src/Miner/*.cpp
  )

  file(GLOB_RECURSE SOURCES src/*.cpp)
  file(GLOB_RECURSE HEADERS src/*.h)
  file(GLOB_RECURSE FORMS src/gui/ui/*.ui)

  if(MSVC)
    include_directories(Platform/Windows)
    set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/Windows/ApplicationEventHandler.cpp)
  elseif(APPLE)
    include_directories(Platform/OSX)
    include_directories(Platform/Posix)
    set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/OSX/ApplicationEventHandler.mm)
    set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/OSX/NotificationMonitor.mm)
    set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/OSX/WalletApplication.mm)
    set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/Posix/System/MemoryMappedFile.cpp)
  else()
    include_directories(Platform/Linux)
    include_directories(Platform/Posix)
    set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/Linux/ApplicationEventHandler.cpp)
    set(SOURCES ${SOURCES} ${CMAKE_SOURCE_DIR}/Platform/Posix/System/MemoryMappedFile.cpp)
  endif()


  set(QRC src/resources.qrc)

  qt5_wrap_ui(UIS ${FORMS})
  qt5_add_resources(RCC ${QRC})

  if (WIN32)
    if (NOT MSVC)
      message(FATAL_ERROR "Only MSVC is supported on this platform")
    endif ()

    if(STATIC)
      foreach(VAR CMAKE_C_FLAGS_DEBUG CMAKE_CXX_FLAGS_DEBUG CMAKE_C_FLAGS_RELWITHDEBINFO CMAKE_CXX_FLAGS_RELWITHDEBINFO CMAKE_C_FLAGS_RELEASE CMAKE_CXX_FLAGS_RELEASE)
        string(REPLACE "/MD" "/MT" ${VAR} "${${VAR}}")
      endforeach()
    endif()

    add_definitions(/D_CRT_SECURE_NO_WARNINGS /D_WIN32_WINNT=0x0600 /DSTATICLIB)
    include_directories(cryptonote/src/Platform/msc)
    link_directories(cryptonote/external/rocksdb)

    set(PLATFORM_DIR Windows)
    set(BUILD_PLATFORM WIN32)
    set(BUILD_RESOURCES src/bbscoinwallet.rc)
    set(QTMAIN Qt5::WinMain)

  elseif (UNIX)
    execute_process(
      COMMAND cmake ${CMAKE_CURRENT_SOURCE_DIR}/cryptonote/external/rocksdb -DWITH_GFLAGS=0 -B${PROJECT_BINARY_DIR}/rocksdb
    )
    set_directory_properties(PROPERTIES ADDITIONAL_MAKE_CLEAN_FILES "${PROJECT_BINARY_DIR}/rocksdb/librocksdb.a")
    add_custom_target(
       rocksdb
       COMMAND make rocksdb
       WORKING_DIRECTORY ${PROJECT_BINARY_DIR}/rocksdb
    )
    add_library(rocksdblib STATIC IMPORTED GLOBAL)
    set_target_properties(rocksdblib PROPERTIES IMPORTED_LOCATION ${PROJECT_BINARY_DIR}/rocksdb/librocksdb.a)
    add_dependencies(rocksdblib rocksdb)

    set(CRYPTONOTE_SOURCES ${CRYPTONOTE_SOURCES} cryptonote/external/miniupnpc/minissdpc.c)

    if (APPLE)
      enable_language(ASM)
      file(GLOB_RECURSE OBJC_SOURCES src/*.mm)
      set(SOURCES ${SOURCES} ${OBJC_SOURCES})
      set(PLATFORM_DIR OSX)
      set(MACOSX_BUNDLE_INFO_STRING "Cryptonote GUI wallet")
      set(MACOSX_BUNDLE_LONG_VERSION_STRING "${VERSION_VERSION}.${VERSION_MINOR}.${VERSION_PATCH}")
      set(MACOSX_BUNDLE_BUNDLE_NAME CryptonoteWallet)
      set(MACOSX_BUNDLE_SHORT_VERSION_STRING "${VERSION_VERSION}.${VERSION_MINOR}.${VERSION_PATCH}")
      set(MACOSX_BUNDLE_BUNDLE_VERSION "$${VERSION_VERSION}.${VERSION_MINOR}.${VERSION_PATCH}")

      find_package(Qt5PrintSupport REQUIRED)

      include_directories(/usr/include/malloc)

      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -maes -std=c++11 -stdlib=libc++")
      set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c11 -maes -D_DARWIN_C_SOURCE")
      set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -framework Cocoa -framework OpenGL -framework CoreFoundation -framework Carbon -framework IOKit -L/usr/lib")

      set(MACOSX_BUNDLE_ICON_FILE bbscoin.icns)
      set(APPLICATION_ICON src/images/bbscoin.icns)
      set_source_files_properties(${APPLICATION_ICON} PROPERTIES MACOSX_PACKAGE_LOCATION "Resources")

      set(BUILD_PLATFORM MACOSX_BUNDLE)
      set(BUILD_RESOURCES ${APPLICATION_ICON})

      GET_TARGET_PROPERTY(QT_LIB_DIR "${Qt5Widgets_LIBRARIES}" LOCATION)
      GET_FILENAME_COMPONENT(QT_LIB_DIR "${QT_LIB_DIR}" PATH)

    else(APPLE)

      set(PLATFORM_DIR Linux)
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -maes -std=c++11")
      set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -maes -std=c11")
    endif (APPLE)
  endif ()

  include_directories(cryptonote/src/Platform/${PLATFORM_DIR})
  file(GLOB PLATFORM_SOURCES cryptonote/src/Platform/${PLATFORM_DIR}/System/*)
  set(CRYPTONOTE_SOURCES ${CRYPTONOTE_SOURCES} ${PLATFORM_SOURCES} ${CRYPTONOTE_PLATFORM})

  add_library(${CRYPTONOTE_LIB} STATIC ${CRYPTONOTE_SOURCES})
  set_target_properties(${CRYPTONOTE_LIB} PROPERTIES COMPILE_DEFINITIONS _GNU_SOURCE)
  target_link_libraries(${CRYPTONOTE_LIB} ${Boost_LIBRARIES} ${Qt5DBus_LIBRARIES} rocksdblib)

  add_executable(${PROJECT_NAME} ${BUILD_PLATFORM} ${BUILD_RESOURCES} ${SOURCES} ${HEADERS} ${UIS} ${RCC})

  set_target_properties(${PROJECT_NAME} PROPERTIES COMPILE_DEFINITIONS _GNU_SOURCE)
  target_link_libraries(${PROJECT_NAME} ${Boost_LIBRARIES} ${QTMAIN} ${CRYPTONOTE_LIB} ${Qt5DBus_LIBRARIES} rocksdblib)

  if (APPLE)
    qt5_use_modules(${PROJECT_NAME} PrintSupport)
  elseif (UNIX)
    target_link_libraries(${PROJECT_NAME} -lpthread)
  elseif (WIN32)
    target_link_libraries(${PROJECT_NAME} Imm32 Iphlpapi Winmm UxTheme d3d9 dxguid Rpcrt4 Wtsapi32 Dwmapi "${_qt5Widgets_install_prefix}/lib/qtharfbuzz.lib" "${_qt5Widgets_install_prefix}/lib/qtlibpng.lib" "${_qt5Widgets_install_prefix}/lib/qtpcre.lib" "${_qt5Widgets_install_prefix}/lib/Qt5EventDispatcherSupport.lib" "${_qt5Widgets_install_prefix}/lib/Qt5OpenGL.lib" "${_qt5Widgets_install_prefix}/lib/Qt5AccessibilitySupport.lib" "${_qt5Widgets_install_prefix}/lib/Qt5ThemeSupport.lib" "${_qt5Widgets_install_prefix}/lib/libGLESv2.lib" "${_qt5Widgets_install_prefix}/lib/libEGL.lib" "${_qt5Widgets_install_prefix}/lib/qtfreetype.lib" "${_qt5Widgets_install_prefix}/lib/Qt5FontDatabaseSupport.lib" "${_qt5Widgets_install_prefix}/lib/preprocessor.lib" "${_qt5Widgets_install_prefix}/lib/translator.lib" "${_qt5Widgets_install_prefix}/plugins/platforms/qwindows.lib")
  endif (APPLE)

  qt5_use_modules(${PROJECT_NAME} Widgets Gui Network)

endif()

# Installation
set(CPACK_PACKAGE_NAME ${CN_PROJECT_NAME})
set(CPACK_PACKAGE_VERSION ${CN_VERSION})
set(CPACK_PACKAGE_VENDOR "BBSCoin")
set(CPACK_PACKAGE_CONTACT "https://www.bbscoin.xyz")
set(CPACK_STRIP_FILES ON)
if (APPLE)
    set(CPACK_GENERATOR DragNDrop)
    install(TARGETS ${PROJECT_NAME} BUNDLE DESTINATION .)
    set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}")
elseif (UNIX)
    find_program(RPMBUILD rpmbuild)

    install(PROGRAMS build/${PROJECT_NAME} DESTINATION bin
    PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)
    install(FILES src/bbscoinwallet.desktop DESTINATION share/applications)
    install(FILES src/images/bbscoin.png DESTINATION share/pixmaps RENAME bbscoin.png)

    if (NOT RPMBUILD)
        set(CPACK_GENERATOR DEB)
        set(CPACK_DEBIAN_PACKAGE_NAME ${CPACK_PACKAGE_NAME})
        set(CPACK_SYSTEM_NAME 64-bit)
        SET(CPACK_PACKAGE_VERSION_MAJOR "${CN_MAJOR_VERSION}")
        SET(CPACK_PACKAGE_VERSION_MINOR "${CN_MINOR_VERSION}")
        SET(CPACK_PACKAGE_VERSION_PATCH "${CN_PATCH_VERSION}")
        set(CPACK_DEBIAN_PACKAGE_MAINTAINER "${CPACK_PACKAGE_VENDOR} <dev@bbscoin.xyz>")
        set(CPACK_DEBIAN_PACKAGE_SECTION Office)
        set(CPACK_DEBIAN_PACKAGE_SHLIBDEPS ON)
        set(CPACK_DEBIAN_PACKAGE_DESCRIPTION "BBSCoin Wallet - A cryptocurrency for exchanging virtual credits from forums and websites")
set(CPACK_PACKAGE_FILE_NAME "${CPACK_DEBIAN_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}${CPACK_SYSTEM_NAME}")
    else()
        set(CPACK_GENERATOR RPM)
        set(CPACK_SYSTEM_NAME x86_64)
        set(CPACK_RPM_PACKAGE_RELEASE ${CPACK_PACKAGE_VERSION})
        set(CPACK_RPM_PACKAGE_LICENSE "MIT")
        set(CPACK_RPM_PACKAGE_GROUP Office)
        set(CPACK_RPM_PACKAGE_REQUIRES "qt5-qtbase >= 5.3.2, qt5-qtbase-gui >= 5.3.2")
        set(CPACK_RPM_PACKAGE_SUMMARY "BBSCoin Wallet")
        set(CPACK_RPM_PACKAGE_DESCRIPTION "BBSCoin Wallet - A cryptocurrency for exchanging virtual credits from forums and websites")
set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME}-${CPACK_PACKAGE_VERSION}${CPACK_SYSTEM_NAME}")
    endif ()
elseif (WIN32)
  install(TARGETS ${PROJECT_NAME} RUNTIME DESTINATION .)
  set(CPACK_GENERATOR ZIP)
endif (APPLE)

include(CPack)