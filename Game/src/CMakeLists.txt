cmake_minimum_required (VERSION 3.25)
project(Lux)

set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_BINARY_DIR})
OPTION(USERDIR_RESOURCES "Allow using local userdirectory resources" ON)
option(SYSTEMWIDE_RESOURCES "Allow system-wide installation of game resources" OFF)
set(MACOSX_BUNDLE_COPYRIGHT "Copyright © 2010-2013 Frictional Games")

if (NOT DEFINED SYSTEMWIDE_RESOURCES_LOCATION)
  set (SYSTEMWIDE_RESOURCES_LOCATION "/usr/local/share/amnesia-amfp")
endif (NOT DEFINED SYSTEMWIDE_RESOURCES_LOCATION)

add_subdirectory(../../HPL2/core engine_libraries)

if(USERDIR_RESOURCES)
    add_definitions(
        -DUSERDIR_RESOURCES
    )
endif()

set(VERSION "1.3.1")

add_subdirectory(game game)

add_custom_target(GameRelease
    DEPENDS Amnesia
)

install (TARGETS Amnesia RUNTIME
         DESTINATION bin)
