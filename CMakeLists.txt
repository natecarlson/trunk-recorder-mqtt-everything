# Copyright 2011 Free Software Foundation, Inc.
#
# This file is part of GNU Radio
#
# GNU Radio is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 3, or (at your option)
# any later version.
#
# GNU Radio is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with GNU Radio; see the file COPYING.  If not, write to
# the Free Software Foundation, Inc., 51 Franklin Street,
# Boston, MA 02110-1301, USA.

########################################################################
# Project setup
########################################################################

cmake_minimum_required(VERSION 3.2)
project(Trunk-Recorder-mqtt CXX C)

# Select the release build type by default to get optimization flags
if(NOT CMAKE_BUILD_TYPE)
   set(CMAKE_BUILD_TYPE "Release")
   message(STATUS "Build type not specified: defaulting to release.")
endif(NOT CMAKE_BUILD_TYPE)
set(CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE} CACHE STRING "")


# Make sure our local CMake Modules path comes first
list(INSERT CMAKE_MODULE_PATH 0 ${CMAKE_SOURCE_DIR}/cmake/Modules)

# Set the version information here
set(VERSION_MAJOR 4)
set(VERSION_API   5)
set(VERSION_ABI   0)
set(VERSION_PATCH git)



########################################################################
# Compiler specific setup
########################################################################
if(CMAKE_COMPILER_IS_GNUCXX AND NOT WIN32)
    #http://gcc.gnu.org/wiki/Visibility
    add_definitions(-fvisibility=hidden)
endif()

if(APPLE)
   set(CMAKE_MACOSX_RPATH ON)
endif()
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib/trunk-recorder")

set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
#SET(CMAKE_SKIP_BUILD_RPATH  FALSE)

########################################################################
# Install directories
########################################################################
include(FindPkgConfig)
find_package(Gnuradio REQUIRED)
#include(GrVersion)


if(NOT CMAKE_MODULES_DIR)
  set(CMAKE_MODULES_DIR lib${LIB_SUFFIX}/cmake)
endif(NOT CMAKE_MODULES_DIR)


if(Gnuradio_VERSION VERSION_LESS "3.8")
    find_package(Volk)
endif()


set(GR_RUNTIME_DIR      bin)
set(GR_LIBRARY_DIR      lib${LIB_SUFFIX})
set(GR_INCLUDE_DIR      include/howto)
set(GR_DATA_DIR         share)
set(GR_PKG_DATA_DIR     ${GR_DATA_DIR}/${CMAKE_PROJECT_NAME})
set(GR_DOC_DIR          ${GR_DATA_DIR}/doc)
set(GR_PKG_DOC_DIR      ${GR_DOC_DIR}/${CMAKE_PROJECT_NAME})
set(GR_CONF_DIR         etc)
set(GR_PKG_CONF_DIR     ${GR_CONF_DIR}/${CMAKE_PROJECT_NAME}/conf.d)
set(GR_LIBEXEC_DIR      libexec)
set(GR_PKG_LIBEXEC_DIR  ${GR_LIBEXEC_DIR}/${CMAKE_PROJECT_NAME})
set(GRC_BLOCKS_DIR      ${GR_PKG_DATA_DIR}/grc/blocks)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})

########################################################################
# Find gnuradio build dependencies
########################################################################
#find_package(GnuradioRuntime)
#find_package(CppUnit)

# To run a more advanced search for GNU Radio and it's components and
# versions, use the following. Add any components required to the list
# of GR_REQUIRED_COMPONENTS (in all caps) and change "version" to the
# minimum API compatible version required.
#
# set(GR_REQUIRED_COMPONENTS RUNTIME BLOCKS FILTER ...)
# find_package(Gnuradio "version")
#if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
#    set(GR_REQUIRED_COMPONENTS ANALOG DIGITAL RUNTIME BLOCKS FILTER PMT)
#    find_package(Gnuradio)
#endif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")


math(EXPR GNURADIO_VERSION "(${Gnuradio_VERSION_MAJOR} / 10) << 20 | (${Gnuradio_VERSION_MAJOR} % 10) << 16 | (${Gnuradio_VERSION_MINOR} / 10) << 12 | (${Gnuradio_VERSION_MINOR} % 10) <<  8 | (${Gnuradio_VERSION_PATCH} / 10) <<  4 | (${Gnuradio_VERSION_PATCH} % 10) <<  0   ")
message(STATUS "GnuRadio Version: " ${GNURADIO_VERSION})



########################################################################
# Find build dependencies
########################################################################
#find_package(GnuradioUHD)
#find_package(GnuradioOsmosdr)
#find_package(LibHackRF)
#find_package(LibUHD)
find_package(OpenSSL REQUIRED)
#find_package(CURL REQUIRED)
find_package(eclipse-paho-mqtt-c REQUIRED)
find_package(PahoMqttCpp REQUIRED)
find_library(trunk-recorder NAMES trunk-recorder/libtrunk-recorder.a REQUIRED)

set(PahoMqttC_LIBRARIES eclipse-paho-mqtt-c::paho-mqtt3as)
set(PahoMqttCpp_LIBRARIES PahoMqttCpp::paho-mqttpp3)
GET_FILENAME_COMPONENT(TRUNK-RECORDER_LIBRARIES "${trunk-recorder}" PATH)

########################################################################
# Setup boost
########################################################################
MESSAGE(STATUS "Configuring Boost C++ Libraries...")

# Although not required on my system, some users have linking issues without
SET(BOOST_REQUIRED_COMPONENTS
    thread
    system
    program_options
    filesystem
    log_setup
    log
    regex
    random
)

if(UNIX AND NOT BOOST_ROOT AND EXISTS "/usr/lib64")
    list(APPEND BOOST_LIBRARYDIR "/usr/lib64") #fedora 64-bit fix
endif(UNIX AND NOT BOOST_ROOT AND EXISTS "/usr/lib64")

set(Boost_ADDITIONAL_VERSIONS
    "1.35.0" "1.35" "1.36.0" "1.36" "1.37.0" "1.37" "1.38.0" "1.38" "1.39.0" "1.39"
    "1.40.0" "1.40" "1.41.0" "1.41" "1.42.0" "1.42" "1.43.0" "1.43" "1.44.0" "1.44"
    "1.45.0" "1.45" "1.46.0" "1.46" "1.47.0" "1.47" "1.48.0" "1.48" "1.49.0" "1.49"
    "1.50.0" "1.50" "1.51.0" "1.51" "1.52.0" "1.52" "1.53.0" "1.53" "1.54.0" "1.54"
    "1.55.0" "1.55" "1.56.0" "1.56" "1.57.0" "1.57" "1.58.0" "1.58" "1.59.0" "1.59"
    "1.60.0" "1.60" "1.61.0" "1.61" "1.62.0" "1.62" "1.63.0" "1.63" "1.64.0" "1.64"
    "1.65.0" "1.65" "1.66.0" "1.66" "1.67.0" "1.67" "1.68.0" "1.68" "1.69.0" "1.69"
    "1.70.0" "1.70" "1.71.0" "1.71" "1.72.0" "1.72" "1.73.0" "1.73" "1.74.0" "1.74"
)

find_package(Boost COMPONENTS ${BOOST_REQUIRED_COMPONENTS} REQUIRED)

if(NOT Boost_FOUND)
    message(FATAL_ERROR "Boost required to build " ${CMAKE_PROJECT_NAME})
endif()

ADD_DEFINITIONS(-DBOOST_ALL_DYN_LINK)
add_definitions(-DGNURADIO_VERSION=${GNURADIO_VERSION})

########################################################################
# Setup the include and linker paths
########################################################################
include_directories(
    ${Boost_INCLUDE_DIRS}
    ${GNURADIO_RUNTIME_INCLUDE_DIRS}
    ${GNURADIO_OSMOSDR_INCLUDE_DIRS}
    ${CMAKE_BINARY_DIR}/lib
    ${CMAKE_BINARY_DIR}/include
    ${CMAKE_SOURCE_DIR}/lib/op25_repeater/lib
    ${CMAKE_SOURCE_DIR}/lib/op25_repeater/include
    ${OPENSSL_INCLUDE_DIR}
    ${CURL_INCLUDE_DIR}
    "lib"
)

link_directories(
    ${Boost_LIBRARY_DIRS}
    ${OPENSSL_ROOT_DIR}/lib
    ${CMAKE_INSTALL_PREFIX}/lib/trunk-recorder
    ${TRUNK-RECORDER_LIBRARIES}
)

set(CMAKE_CXX_FLAGS_DEBUG "-Wall -Wno-unused-local-typedef -Wno-deprecated-declarations -Wno-error=deprecated-declarations -g3")

SET(CMAKE_CXX_STANDARD 11)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread -Wno-narrowing -fvisibility=hidden -fPIC")

include(GNUInstallDirs)
include_directories("/usr/local/include")

add_library(mqtt_status_plugin
MODULE
  mqtt_status_plugin.cc
)

target_link_libraries(mqtt_status_plugin ${trunk-recorder} ssl crypto ${PahoMqttC_LIBRARIES} ${PahoMqttCpp_LIBRARIES} ${Boost_LIBRARIES} ${GNURADIO_PMT_LIBRARIES} ${GNURADIO_RUNTIME_LIBRARIES} ${GNURADIO_FILTER_LIBRARIES} ${GNURADIO_DIGITAL_LIBRARIES} ${GNURADIO_ANALOG_LIBRARIES} ${GNURADIO_AUDIO_LIBRARIES} ${GNURADIO_UHD_LIBRARIES} ${UHD_LIBRARIES} ${GNURADIO_BLOCKS_LIBRARIES} ${GNURADIO_OSMOSDR_LIBRARIES}  ${LIBOP25_REPEATER_LIBRARIES} gnuradio-op25_repeater)

if(NOT Gnuradio_VERSION VERSION_LESS "3.8")
    target_link_libraries(mqtt_status_plugin) 
endif()

install(TARGETS mqtt_status_plugin LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib/trunk-recorder)