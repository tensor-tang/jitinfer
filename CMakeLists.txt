# This file is part of the JITLIB distribution (https://github.com/tensor-tang).
# Copyright (c) 2018 Tensor Tang.
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, version 3.
#
# This program is distributed in the hope that it will be useful, but
# WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
# General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program. If not, see <http://www.gnu.org/licenses/>.

cmake_minimum_required(VERSION 3.5)

if(WIN32 OR APPLE)
  message(FATAL "Windows or Mac is not supported yet.")
endif()

# CMAKE_BUILD_TYPE
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING
      "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel"
      FORCE)
endif()

set(PROJECT_NAME "JitLib")
set(PROJECT_VERSION "0.1")
set(LIB_NAME jitlib)
set(PROJECT_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(PROJECT_BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR})
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
set(THIRD_PARTY_PATH "${CMAKE_BINARY_DIR}/third_party" CACHE STRING
  "A path setting third party libraries download & build directories.")
set(THIRD_PARTY_BUILD_TYPE Release)

project(${PROJECT_NAME} C CXX)
message(STATUS "CXX compiler: ${CMAKE_CXX_COMPILER}, version: "
        "${CMAKE_CXX_COMPILER_ID} ${CMAKE_CXX_COMPILER_VERSION}")
message(STATUS "C compiler: ${CMAKE_C_COMPILER}, version: "
        "${CMAKE_C_COMPILER_ID} ${CMAKE_C_COMPILER_VERSION}")
################################ Configurations #######################################
option(WITH_VNNI          "Compile with AVX512 VNNI support"        ${VNNI_FOUND})
option(WITH_GTEST         "Compile with gtest"                                 ON)
option(WITH_VERBOSE       "Compile with VERBOSE for profiling"                 ON)

########################################################################################
include(system)                # gcc flags
include(simd)                # used to check if is supported AVX512
include(external/mklml)      # download mklml package and use iomp
include(external/xbyak)      # download xbyak
include(external/gtest)      # download, build, install gtest
#include(util)                # some utility functions

if(NOT ${AVX512_FOUND})
  message(FATAL "AVX512 is not found! Only support on AVX512 yet!")
endif()

if(WITH_VERBOSE)
  add_definitions(-DWITH_VERBOSE)
endif()

add_subdirectory(src)
add_subdirectory(test)

include_directories("${PROJECT_SOURCE_DIR}")

set(CMAKE_CXX_FLAGS_DEBUG "-ggdb3 -O0" CACHE STRING "Disable optimize when debug")
set(CMAKE_C_FLAGS_DEBUG "-ggdb3 -O0"  CACHE STRING "Disable optimize when debug")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O3 -g -DNDEBUG")
set(CMAKE_C_FLAGS_RELWITHDEBINFO "-O3 -g -DNDEBUG")