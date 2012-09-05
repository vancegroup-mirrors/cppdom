# CMake-based build system for cppdom
# 2009-2011 Ryan Pavlik <rpavlik@iastate.edu>
# http://academic.cleardefinition.com/
# Iowa State University HCI Graduate Program/VRAC

cmake_minimum_required(VERSION 2.8.8)

# Set package properties
project(cppdom)

set(CPACK_PACKAGE_VERSION_MAJOR "1")
set(CPACK_PACKAGE_VERSION_MINOR "2")
set(CPACK_PACKAGE_VERSION_PATCH "0")
set(CPACK_PACKAGE_VERSION
    "${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}")
set(VERSION_UNDERSCORES "${CPACK_PACKAGE_VERSION_MAJOR}_${CPACK_PACKAGE_VERSION_MINOR}_${CPACK_PACKAGE_VERSION_PATCH}")

###
# Set up options
###

# TODO this is not ideal/complete
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(arch "x86_64")
    set(ARCH_DIR lib64)
else()
    set(arch "x86")
    set(ARCH_DIR lib)
endif()

# Define directories
set(BIN_DIR bin)

if(WIN32)
    set(LIB_DIR bin)
    set(INCLUDE_DIR include)
else()
    set(LIB_DIR ${ARCH_DIR})
    set(INCLUDE_DIR include/cppdom-${CPACK_PACKAGE_VERSION})
endif()

set(FLAGPOLL_INFO_DIR ${ARCH_DIR}/flagpoll)

# Create flagpoll file
set(provides "cppdom")
set(version "${CPACK_PACKAGE_VERSION}")
set(prefix "\${fp_file_cwd}/../..")
set(exec_prefix "\${prefix}")
set(includedir "\${fp_file_cwd}/../../${INCLUDE_DIR}")
set(libdir "\${fp_file_cwd}/../../${ARCH_DIR}")

if(CMAKE_COMPILER_IS_GNUCXX OR ( CMAKE_CXX_COMPILER_ID STREQUAL "Clang" ) )
    set(cppdom_cxxflags "-Wno-deprecated")
    set(include_path_flag "-I")
    set(cppdom_libs "-lcppdom-${VERSION_UNDERSCORES}")
    set(lib_path_flag "-L")
elseif(MSVC)
    set(include_path_flag "/I")
    set(lib_path_flag "/libpath:")
else()
    message(FATAL_ERROR "Compiler not recognized - can't generate values for Flagpoll file")
endif()

configure_file(cppdom.fpc.in "${CMAKE_CURRENT_BINARY_DIR}/cppdom.fpc" @ONLY)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/cppdom.fpc"
        DESTINATION ${FLAGPOLL_INFO_DIR}
        COMPONENT Development)

install(FILES cppdom-config
        DESTINATION ${BIN_DIR}
        COMPONENT Development)

###
# Perform build configuration of dependencies
###
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

include(CTest)

option( BUILD_SPIRIT_PARSER "Set to ON to build the SpiritParser based on the Boost Spirit library." OFF )
if(BUILD_SPIRIT_PARSER)
    find_package(Boost)
endif(BUILD_SPIRIT_PARSER)

###
# Build the project
###

include_directories(.)
add_subdirectory(cppdom)

add_subdirectory(test)

###
# Set packaging options (for CPack)
###

# Choose desired package generators
if(APPLE)
    set(CPACK_GENERATOR DragNDrop)
    set(CPACK_SOURCE_GENERATOR ZIP)
elseif(WIN32)
    set(CPACK_SOURCE_GENERATOR ZIP)
else()
    set(CPACK_SOURCE_GENERATOR TARGZ)
endif()

# Include the packaging system now that we have it all set up
include(CPack)

###
# End Packaging
###

# Install the CMake find script for cppdom
install( FILES "cmake/Findcppdom.cmake"
         DESTINATION "share/${CMAKE_PROJECT_NAME}/extra"
         COMPONENT Development )

#
# Create and install all of the auto find tools
# =============================================

# Add all targets to the build-tree export set
export(TARGETS cppdom 
    FILE "${PROJECT_BINARY_DIR}/${ARCH_DIR}/cppdomLibraryDepends.cmake")

# Export the package for use from the build-tree
# (this registers the build-tree with a global CMake-registry)
export(PACKAGE cppdom)

# Create a cppdomBuildTreeSettings.cmake file for the use from the build tree
configure_file(cppdomBuildTreeSettings.cmake.in
    "${PROJECT_BINARY_DIR}/${ARCH_DIR}/cppdomBuildTreeSettings.cmake" @ONLY)

configure_file(cppdomConfigVersion.cmake.in
    "${PROJECT_BINARY_DIR}/${ARCH_DIR}/cppdomConfigVersion.cmake" @ONLY)
configure_file(Usecppdom.cmake.in
    "${PROJECT_BINARY_DIR}/${ARCH_DIR}/Usecppdom.cmake" @ONLY)

# Install the export set for use with the install-tree
install(EXPORT cppdom-targets DESTINATION ${ARCH_DIR})

include( CMakePackageConfigHelpers )
configure_package_config_file( 
    cppdomConfig.cmake.in ${CMAKE_CURRENT_BINARY_DIR}/${ARCH_DIR}/cppdomConfig.cmake
    INSTALL_DESTINATION ${ARCH_DIR}
    PATH_VARS INCLUDE_DIR )

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${ARCH_DIR}/cppdomConfig.cmake 
              ${CMAKE_CURRENT_BINARY_DIR}/${ARCH_DIR}/cppdomConfigVersion.cmake
              ${CMAKE_CURRENT_BINARY_DIR}/${ARCH_DIR}/Usecppdom.cmake
        DESTINATION ${ARCH_DIR} )
