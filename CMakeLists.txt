# Copyright (c) 2014 K. Kumar (me@kartikkumar.com)
# Distributed under the Apache License, Version 2.0
# See accompanying file COPYING or copy at http://opensource.org/licenses/Apache-2.0

cmake_minimum_required(VERSION 2.6)

# Change this section to correspond with your project info.
project(SGP4 C CXX)
set(MYPROJ_DESC                              "SGP4/SDP4 models implemented in C++")
set(MYPROJ_UPPER                             "SGP4")
set(MYPROJ_LIB                               "sgp4")
set(${CMAKE_PROJECT_NAME}_VENDOR_NAME        "deorbit")
set(${CMAKE_PROJECT_NAME}_VENDOR_CONTACT     "https://github.com/deorbit")

set(${CMAKE_PROJECT_NAME}_VERSION_MAJOR      0)
set(${CMAKE_PROJECT_NAME}_VERSION_MINOR      1)
set(${CMAKE_PROJECT_NAME}_VERSION_PATCH      0)
set(MYPROJ_VERSION
  "${${CMAKE_PROJECT_NAME}_VERSION_MAJOR}.${${CMAKE_PROJECT_NAME}_VERSION_MINOR}.${${CMAKE_PROJECT_NAME}_VERSION_PATCH}")

set(MYPROJ_PATH                              "${CMAKE_CURRENT_SOURCE_DIR}")
set(MYINC_PATH                               "${MYPROJ_PATH}/libsgp4")
set(MYSRC_PATH                               "${MYPROJ_PATH}/libsgp4")

OPTION(BUILD_DOCS                            "Build docs" OFF)

set(MYPROJ_SRC  
  "${MYSRC_PATH}/Eci.cpp"
  "${MYSRC_PATH}/Observer.cpp"
  "${MYSRC_PATH}/OrbitalElements.cpp"
  "${MYSRC_PATH}/SGP4.cpp"
  "${MYSRC_PATH}/SolarPosition.cpp"
  "${MYSRC_PATH}/Tle.cpp"
  "${MYSRC_PATH}/Util.cpp"
)

# Set CMake build-type. If it not supplied by the user, the default built type is "Release". 
if((NOT CMAKE_BUILD_TYPE) OR (CMAKE_BUILD_TYPE STREQUAL "Release"))
    message(STATUS "WARNING: building release version!")
    set(CMAKE_BUILD_TYPE Release)
elseif(CMAKE_BUILD_TYPE STREQUAL "Debug")
    message(STATUS "WARNING: building debug version!")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -pg")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pg")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -pg")
endif((NOT CMAKE_BUILD_TYPE) OR (CMAKE_BUILD_TYPE STREQUAL "Release"))

# Set platform-specific compiler flags.
if(WIN32)
  if(MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHsc /Ox /W3 /FC -D_SCL_SECURE_NO_WARNINGS")
    # We need to force MSVC to use static runtime: 
    # (from http://www.cmake.org/Wiki/CMake_FAQ#Dynamic_Replace).
    foreach(flag_var
      CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
      CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO)
      # Find all dynamic runtime (MD) references and replace with static (MT)
      if(${flag_var} MATCHES "/MD")
        string(REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
      endif(${flag_var} MATCHES "/MD")
    endforeach(flag_var)
  else(MSVC)
    set(CMAKE_CXX_FLAGS 
      "${CMAKE_CXX_FLAGS} -Wall -Woverloaded-virtual -Wold-style-cast -Wnon-virtual-dtor")
  endif(MSVC)
else(WIN32)
  set(CMAKE_CXX_FLAGS 
    "${CMAKE_CXX_FLAGS} -Wall -Woverloaded-virtual -Wold-style-cast -Wnon-virtual-dtor")
endif(WIN32)

# Allows for external dependencies to be built by CMake.
include(ExternalProject)

include_directories(AFTER "${MYINC_PATH}")

if(NOT MYLIB_PATH)
  set(MYLIB_PATH "${MYSRC_PATH}")
endif(NOT MYLIB_PATH)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${MYLIB_PATH})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${MYLIB_PATH})
add_library(${MYPROJ_LIB} ${MYPROJ_SRC})

if(BUILD_DOCS)
  find_package(Doxygen)

  if(NOT DOXYGEN_FOUND)
    message(FATAL_ERROR 
            "Doxygen is needed to build the documentation. Please make sure it is installed.")
  endif(NOT DOXYGEN_FOUND)

  configure_file(${MYPROJ_PATH}/Doxyfile.in 
                 ${PROJECT_BINARY_DIR}/Doxyfile @ONLY IMMEDIATE)

  # Add a custom target to run Doxygen whenever the project is built.
  # If you do NOT want the documentation to be generated EVERY time you build the project
  # then leave out the 'ALL' keyword from the command below.
  add_custom_target(docs ALL COMMAND ${DOXYGEN_EXECUTABLE} 
                    ${PROJECT_BINARY_DIR}/Doxyfile 
                    SOURCES ${PROJECT_BINARY_DIR}/Doxyfile)
endif(BUILD_DOCS)

# Install header files and library. 
# Destination is set by CMAKE_INSTALL_PREFIX and defaults to usual locations, unless overridden by
# user.
install(DIRECTORY ${MYINC_PATH} 
        DESTINATION include
        FILES_MATCHING PATTERN "*.h")
install(TARGETS ${MYPROJ_LIB} DESTINATION lib)

# Set up packager.
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "${CMAKE_PROJECT_NAME}")
set(CPACK_PACKAGE_VENDOR "${CMAKE_PROJECT_NAME}_VENDOR_NAME")
set(CPACK_PACKAGE_DESCRIPTION_FILE "${MYPROJ_PATH}/README.md")
set(CPACK_RESOURCE_FILE_LICENSE "${MYPROJ_PATH}/COPYING")
set(CPACK_PACKAGE_VERSION_MAJOR "${${CMAKE_PROJECT_NAME}_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${${CMAKE_PROJECT_NAME}_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${${CMAKE_PROJECT_NAME}_VERSION_PATCH}")
set(CPACK_PACKAGE_INSTALL_DIRECTORY "${CMAKE_PROJECT_NAME}")
if(WIN32 AND NOT UNIX)
  # There is a bug in NSI that does not handle full unix paths properly. Make
  # sure there is at least one set of four (4) backlasshes.
  set(CPACK_NSIS_DISPLAY_NAME "${CPACK_PACKAGE_INSTALL_DIRECTORY}")
  set(CPACK_NSIS_CONTACT "${CMAKE_PROJECT_NAME}_VENDOR_CONTACT")
  set(CPACK_NSIS_MODIFY_PATH ON)
else(WIN32 AND NOT UNIX)
endif(WIN32 AND NOT UNIX)
include(CPack)
