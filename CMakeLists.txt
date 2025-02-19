#set(Qt6Core_DIR "/opt/Qt6.6.0/5.6/gcc_64/lib/cmake/Qt6Core")
#set(Qt6Test_DIR "/opt/Qt6.6.0/5.6/gcc_64/lib/cmake/Qt6Test")
#set(Qt6_DIR "/opt/Qt6.6.0/5.6/gcc_64/lib/cmake/Qt6Core")
#set(QT_QMAKE_EXECUTABLE "/opt/Qt6.6.0/5.6/gcc_64/bin/qmake")

project(qmsgpack)

cmake_minimum_required(VERSION 3.1.0)
#set (CMAKE_CXX_STANDARD 20)

set(CMAKE_INSTALL_NAME_DIR ${LIB_INSTALL_DIR})
set(CMAKE_INCLUDE_CURRENT_DIR ON)

# build type
if ("${CMAKE_BUILD_TYPE}" MATCHES "^Rel.*")
  add_definitions("-DQT_NO_DEBUG_OUTPUT")
endif ("${CMAKE_BUILD_TYPE}" MATCHES "^Rel.*")

# 'd' postfix on windows if debug
if (WIN32)
	set(CMAKE_DEBUG_POSTFIX "d")
endif (WIN32)

option (QT4_BUILD "BUild with Qt4")
if (NOT QT4_BUILD)
        find_package(Qt6Core QUIET)
endif ()

if (Qt6Core_FOUND)
        message("Qt6 found")
        include_directories(${Qt6Core_INCLUDE_DIRS})
        add_definitions(${Qt6Core_DEFINITIONS})
        # Find includes in corresponding build directories
        set(CMAKE_INCLUDE_CURRENT_DIR ON)
        # Instruct CMake to run moc automatically when needed.
        set(CMAKE_AUTOMOC ON)
        set(PC_Requires "Qt6Core")
else ()
        message("Qt6 not found, searching for Qt4")
        find_package(Qt4 REQUIRED)
        include(${QT_USE_FILE})
        # Find includes in corresponding build directories
        set(CMAKE_INCLUDE_CURRENT_DIR ON)
        # Instruct CMake to run moc automatically when needed.
        set(CMAKE_AUTOMOC ON)
        set(PC_Requires "QtCore")
endif ()

option (WITH_GUI_TYPES "Build with support for QtGui types")
if (WITH_GUI_TYPES)
        if (QT4_BUILD)
                find_package(Qt4 QTGUI)
        else ()
                find_package(Qt6Gui QUIET)
        endif ()
endif ()

option (WITH_LOCATION_TYPES "Build with support for QtLocation types")
if (WITH_LOCATION_TYPES)
        find_package(Qt6Location QUIET)
endif ()

if (Qt6Location_FOUND)
        message("Qt6Location found")
        include_directories(${Qt6Location_INCLUDE_DIRS})
        add_definitions(${Qt6Location_DEFINITIONS})
else ()
        message("Qt6Location not found")
endif ()

if (NOT WIN32)
	set(QT_DONT_USE_QTGUI TRUE)
endif ()

#add extra search paths for libraries and includes
set(LIB_SUFFIX "" CACHE STRING "Define suffix of directory name (32/64)" )
set(LIB_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/lib${LIB_SUFFIX}" CACHE STRING "Directory where lib will install")
set(INCLUDE_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/include" CACHE PATH "The directory the headers are installed in")
set(CMAKECONFIG_INSTALL_DIR "${LIB_INSTALL_DIR}/cmake/${CMAKE_PROJECT_NAME}" CACHE PATH "Directory where to install qmsgpack.cmake")

configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
  IMMEDIATE @ONLY)

set(QMSGPACK_MAJOR	"0")
set(QMSGPACK_MINOR	"1")
set(QMSGPACK_VERSION	"0")

set(MSGPACK_QT_LIB_VERSION_STRING "${QMSGPACK_MAJOR}.${QMSGPACK_MINOR}.${QMSGPACK_VERSION}")

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")

add_subdirectory(src)
if (BUILD_TESTS)
	enable_testing()
	add_subdirectory(tests)
endif ()

# install(EXPORT qmsgpack-export DESTINATION ${CMAKECONFIG_INSTALL_DIR} FILE MsgPackQtTargets.cmake)

file(RELATIVE_PATH relInstallDir ${CMAKE_INSTALL_PREFIX}/$CMAKECONFIG_INSTALL_DIR} ${CMAKE_INSTALL_PREFIX})

add_custom_target(uninstall
  "${CMAKE_COMMAND}" -P "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake")
