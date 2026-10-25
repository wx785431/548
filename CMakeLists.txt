# Copyright (C) 2011-2016 Project SkyFire <http://www.projectskyfire.org/>
# Copyright (C) 2008-2016 TrinityCore <http://www.trinitycore.org/>
#
# This file is free software; as a special exception the author gives
# unlimited permission to copy and/or distribute it, with or without
# modifications, as long as this notice is preserved.
#
# This program is distributed in the hope that it will be useful, but
# WITHOUT ANY WARRANTY, to the extent permitted by law; without even the
# implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

# Before project or language initialization

# cmake_minimum_required() must be the first command of the top-level file
cmake_minimum_required(VERSION 3.16)

# Set projectname (must be done AFTER setting configurationtypes)
project(Pandaria_5.4.8)

# CMake policies (can not be handled elsewhere)
# NOTE: The old code forced CMP0005/CMP0043 to OLD here. CMake 4.x no longer
# supports the OLD behaviour of those policies and aborts the configure step
# ("Policy CMP0005 may not be set to OLD behavior"), so the settings have been
# dropped. cmake_minimum_required(3.16) already selects NEW behaviour for both.

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_EXTENSIONS OFF)
# add this options before PROJECT keyword
set(CMAKE_DISABLE_SOURCE_CHANGES ON)
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)

# Set RPATH-handing (CMake parameters)
set(CMAKE_SKIP_BUILD_RPATH 0)
set(CMAKE_BUILD_WITH_INSTALL_RPATH 0)
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH 1)
	   
# set macro-directory
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/macros")

# build in Release-mode by default if not explicitly set
if( NOT CMAKE_BUILD_TYPE )
  set(CMAKE_BUILD_TYPE "Release")
endif()

include(CheckSymbolExists)
include(CheckCXXSourceRuns)
include(CheckIncludeFiles)

# set default buildoptions and print them
include(cmake/options.cmake)

# turn off PCH totally if enabled (hidden setting, mainly for devs)
if( NOPCH )
  set(USE_COREPCH 0)
  set(USE_SCRIPTPCH 0)
endif()

include(CheckPlatform)

# basic packagesearching and setup (further support will be needed, this is a preliminary release!)
set(ACE_EXPECTED_VERSION 5.8.3)

find_package(ACE REQUIRED)
find_package(OpenSSL REQUIRED)
find_package(Threads REQUIRED)

if( NOT USE_MYSQL_SOURCES )
  find_package(MySQL REQUIRED)
endif()

#Debug compile time
set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CMAKE_COMMAND} -E time")
set_property(GLOBAL PROPERTY RULE_LAUNCH_LINK "${CMAKE_COMMAND} -E time")

if( UNIX )
  find_package(Readline)
  find_package(ZLIB)
  find_package(BZip2)
endif()

if(WIN32)
  add_definitions(-DBOOST_DATE_TIME_NO_LIB -DBOOST_REGEX_NO_LIB -DBOOST_CHRONO_NO_LIB -DBOOST_SERIALIZATION_NO_LIB)
  set(BOOST_DEBUG OFF)
  if(DEFINED ENV{BOOST_ROOT})
    set(BOOST_ROOT $ENV{BOOST_ROOT})
    list(APPEND BOOST_LIBRARYDIR
      ${BOOST_ROOT}/lib${PLATFORM}-msvc-14.2 )
  endif()


  set(Boost_USE_STATIC_LIBS        ON)
  set(Boost_USE_MULTITHREADED      ON)
  set(Boost_USE_STATIC_RUNTIME     OFF)
else()
  set(Boost_USE_STATIC_LIBS ON)
endif()

find_package(Boost COMPONENTS system locale thread serialization filesystem program_options iostreams regex
    REQUIRED
)

# Find revision ID and hash of the sourcetree
include(cmake/genrev.cmake)

# print out the results before continuing
include(cmake/showoptions.cmake)

# add dependencies
add_subdirectory(dep)

# add core sources
add_subdirectory(src)
