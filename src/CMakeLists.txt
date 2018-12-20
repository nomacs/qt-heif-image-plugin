cmake_minimum_required(VERSION 3.5)  # lowest version tried

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# load paths from the user file if exists
if (EXISTS ${CMAKE_SOURCE_DIR}/CMakeUserPaths.cmake)
	include(${CMAKE_SOURCE_DIR}/CMakeUserPaths.cmake)
	
	 if(NOT QT_QMAKE_EXECUTABLE)
		find_program(QT_QMAKE_EXECUTABLE NAMES "qmake" "qmake-qt5" "qmake.exe")
	 endif()
	 if(NOT QT_QMAKE_EXECUTABLE)
		message(FATAL_ERROR "you have to set the path to the Qt5 qmake executable")
	 endif()
	 
	 message(STATUS "QMake found: ${QT_QMAKE_EXECUTABLE}")
	 get_filename_component(QT_QMAKE_PATH ${QT_QMAKE_EXECUTABLE} PATH)
	 
	set(QT_ROOT ${QT_QMAKE_PATH}/)
	set(CMAKE_PREFIX_PATH ${CMAKE_PREFIX_PATH} ${QT_QMAKE_PATH}/../lib/cmake/Qt5)
	
	message(STATUS "using user paths, new prefix path: ${CMAKE_PREFIX_PATH}")
elseif(MSVC)
	message(WARNING "Could not find CMakeUserPaths.cmake - please use this file to specify your install directories (see CMakeUserPathsGit.cmake)")
endif()

# make release build, if not specified
# (from https://blog.kitware.com/cmake-and-the-default-build-type/)
set(default_build_type "RelWithDebInfo")
set(CMAKE_DEBUG_POSTFIX "d")

if (NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE STRING "Build type" FORCE)
  set_property(
    CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
    "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif ()

# compiler flags
# TODO: separate GCC and Clang warnings; add more
if (!MSVC)
	set(
		  CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} \
		  -Wall \
		  -Wextra \
		  -Wshadow \
		  -Wformat-nonliteral \
		  -Wformat-security \
		  -Wnon-virtual-dtor \
		  ")
  
	set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -Og")
	set(sanitizer_flags "-fsanitize=address -fsanitize=undefined")

endif()


set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ${sanitizer_flags}")
set(CMAKE_MODULE_LINKER_FLAGS_DEBUG
    "${CMAKE_MODULE_LINKER_FLAGS_DEBUG} ${sanitizer_flags}")

#
# third-party libs
#

# qt
find_package(Qt5 COMPONENTS Core Gui REQUIRED)
add_definitions(-DQT_NO_KEYWORDS)
set(CMAKE_AUTOMOC ON)

# libheif
if (!MSVC)
	find_package(PkgConfig)
	pkg_check_modules(libheif REQUIRED libheif>=1.1)
else()

	find_package(libheif)
	include_directories( ${libheif_INCLUDE_DIR})

	if(!LIBHEIF_FOUND)
		message(FATAL_ERROR "I could not find libheif which is needed. You should add its path to the CMAKE_PREFIX_PATHS")
	endif()

endif()

#
# project source
#

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(sources main.cpp qheifhandler.cpp)

add_library(qheif MODULE ${sources})

target_link_libraries(
  qheif
  PRIVATE
  Qt5::Gui
	${libheif_LIBRARIES}
)

#
# installation
#

# Use qmake to find plugin dir (adapted from lxqt-qtplugin)
get_target_property(
  QT_QMAKE_EXECUTABLE ${Qt5Core_QMAKE_EXECUTABLE} IMPORTED_LOCATION)

if (NOT QT_QMAKE_EXECUTABLE)
  message(FATAL_ERROR "qmake is not found.")
endif ()

execute_process(
  COMMAND ${QT_QMAKE_EXECUTABLE} -query QT_INSTALL_PLUGINS
  OUTPUT_VARIABLE QT_PLUGINS_DIR
  OUTPUT_STRIP_TRAILING_WHITESPACE)

if (QT_PLUGINS_DIR)
  message(STATUS "Qt5 plugin directory: " "${QT_PLUGINS_DIR}")
else ()
  message(FATAL_ERROR "Qt5 plugin directory cannot be detected.")
endif ()

# Prefix with DESTDIR if available to allow packaging
if (ENV{DESTDIR} AND NOT ENV{DESTDIR} STREQUAL "")
  set(plugins_dir "$ENV{DESTDIR}${QT_PLUGINS_DIR}")
else ()
  set(plugins_dir "${QT_PLUGINS_DIR}")
endif ()

install(
  TARGETS qheif
  LIBRARY DESTINATION "${plugins_dir}/imageformats")

# vim:sw=2
