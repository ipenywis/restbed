# Copyright 2013-2016, Corvusoft Ltd, All Rights Reserved.

project( "restbed" )

cmake_minimum_required( VERSION 2.8.10 )

#
# Build Options
#
option( BUILD_SHARED   "Build shared library."              OFF )
option( BUILD_EXAMPLES "Build examples applications."       OFF )
option( BUILD_TESTS    "Build all available test suites."   OFF )
option( BUILD_SSL      "Build secure socket layer support."  ON )

#
# Configuration
#
include( "${CMAKE_SOURCE_DIR}/cmake/configuration.cmake" )

message( "   ${Blue}Copyright 2013-2016, Corvusoft Ltd, All Rights Reserved.${Reset}" )

#
# Dependencies
#
set( CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/modules" )

find_package( asio REQUIRED )
include_directories( SYSTEM ${asio_INCLUDE} )

if ( BUILD_SSL )
    find_package( openssl REQUIRED )
    include_directories( SYSTEM ${ssl_INCLUDE} )
endif ( )

#
# Build
#
include( "${CMAKE_SOURCE_DIR}/cmake/manifest.cmake" )

include_directories( ${INCLUDE_DIR} )

add_library( ${PROJECT_NAME} ${MANIFEST} )

if ( BUILD_SSL )
    target_link_libraries( ${PROJECT_NAME} LINK_PRIVATE ${ssl_LIBRARY} ${crypto_LIBRARY} )
else ( )
    target_link_libraries( ${PROJECT_NAME} )
endif ( )

if ( BUILD_EXAMPLES )
    find_package( pam )
    find_package( syslog )
    add_subdirectory( "${CMAKE_SOURCE_DIR}/example" )
endif ( )

if ( BUILD_TESTS )
    find_package( catch REQUIRED )

    enable_testing( )
    add_subdirectory( "${CMAKE_SOURCE_DIR}/test/unit" )
    add_subdirectory( "${CMAKE_SOURCE_DIR}/test/acceptance" )
    add_subdirectory( "${CMAKE_SOURCE_DIR}/test/regression" )
    add_subdirectory( "${CMAKE_SOURCE_DIR}/test/integration" )
endif ( )

#
# Install
#
include( "${CMAKE_SOURCE_DIR}/cmake/artifacts.cmake" )

install( FILES "${INCLUDE_DIR}/${PROJECT_NAME}" DESTINATION "include" )
install( FILES ${ARTIFACTS} DESTINATION "include/corvusoft/${PROJECT_NAME}" )
install( TARGETS ${PROJECT_NAME} LIBRARY DESTINATION "library" ARCHIVE DESTINATION "library" COMPONENT library )
