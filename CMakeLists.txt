cmake_minimum_required(VERSION 2.8)
project(Performous CXX C)
set(PROJECT_VERSION "0.8")
set(BUILD_SHARED_LIBS OFF)

# Avoid source tree pollution
if(CMAKE_SOURCE_DIR STREQUAL CMAKE_BINARY_DIR)
	message(FATAL_ERROR "In-source builds are not permitted. Make a separate folder for building:\nmkdir build; cd build; cmake ..\nBefore that, remove the files already created:\nrm -rf CMakeCache.txt CMakeFiles")
endif(CMAKE_SOURCE_DIR STREQUAL CMAKE_BINARY_DIR)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")

# Add a sensible build type default and warning because empty means no optimization and no debug info.
if(NOT CMAKE_BUILD_TYPE)
	message("WARNING: CMAKE_BUILD_TYPE is not defined!\n         Defaulting to CMAKE_BUILD_TYPE=RelWithDebInfo. Use ccmake to set a proper value.")
	SET(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Choose the type of build, options are: None Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif(NOT CMAKE_BUILD_TYPE)

if(CMAKE_BUILD_TYPE STREQUAL "Release")
	add_definitions("-DNDEBUG")
endif()

# Enable C++11 support and other compiler flags...
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
	execute_process(COMMAND ${CMAKE_CXX_COMPILER} -dumpversion OUTPUT_VARIABLE GCC_VERSION)
	message("Detected GCC version ${GCC_VERSION}")
	if (GCC_VERSION VERSION_GREATER 4.7 OR GCC_VERSION VERSION_EQUAL 4.7)
		set(CMAKE_CXX_FLAGS "-std=c++11 -Wall -Wextra -fcx-limited-range ${CMAKE_CXX_FLAGS}")
	elseif (GCC_VERSION VERSION_GREATER 4.6 OR GCC_VERSION VERSION_EQUAL 4.6)
		set(CMAKE_CXX_FLAGS "-std=c++0x -Wall -Wextra -fcx-limited-range ${CMAKE_CXX_FLAGS}")
		add_definitions("-Doverride=")  # Hack: GCC 4.6 doesn't understand C++11 "override" keyword
	else()
		message(FATAL_ERROR "Your GCC compiler does not support c++11, please install at least gcc 4.6")
	endif()
endif()

if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
	set(CMAKE_CXX_FLAGS "-std=c++11 -Wall -Wextra ${CMAKE_CXX_FLAGS}")
endif()

if(WIN32)
	set(CMAKE_CXX_FLAGS "-DWIN32_LEAN_AND_MEAN ${CMAKE_CXX_FLAGS}")
	set(SHARE_INSTALL_DEFAULT ".")
	SET(LOCALE_DIR_DEFAULT "locale")
else()
	set(SHARE_INSTALL_DEFAULT "share/games/performous")
	SET(LOCALE_DIR_DEFAULT "share/locale")
endif()

set(SHARE_INSTALL "${SHARE_INSTALL_DEFAULT}" CACHE STRING "Data file install path. Must be a relative path (from CMAKE_INSTALL_PREFIX), with no trailing slash.")
mark_as_advanced(SHARE_INSTALL)

set(LOCALE_DIR "${LOCALE_DIR_DEFAULT}" CACHE STRING "Locale file install path. Must be a relative path (from CMAKE_INSTALL_PREFIX), with no trailing slash.")
mark_as_advanced(LOCALE_DIR)

find_package(Gettext)
find_package(Msgfmt)
if(Gettext_FOUND AND Msgfmt_FOUND)
	message(STATUS "Localization enabled: Building and installing .mo files")
	add_subdirectory(lang)
else()
	message(STATUS "Localization disabled: Gettext or Msgfmt missing")
endif()

add_subdirectory(data)
add_subdirectory(game)

option(ENABLE_TOOLS "Enable building of song extraction utilities for SingStar and Guitar Hero." ON)
if(ENABLE_TOOLS)
	add_subdirectory(tools)
endif()

add_subdirectory(docs)

if(WIN32)
	option(ENABLE_VERSIONING "Add version number information. This allows adding version information to the EXE. Be careful, in beta." OFF)
	install(FILES win32/ConfigureSongDirectory.bat DESTINATION .)
endif()

