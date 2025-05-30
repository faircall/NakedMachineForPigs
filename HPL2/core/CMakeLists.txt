cmake_minimum_required (VERSION 3.25)
project(HPL2)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  set(LINUX ON)
  set(LINUX ON PARENT_SCOPE)
endif()

set(HPL2_VERSION_MAJOR 2)
set(HPL2_VERSION_MINOR 0)

add_subdirectory(../dependencies/OALWrapper OALWrapper)
add_subdirectory(../dependencies/AngelScript AngelScript)
add_subdirectory(../dependencies/Newton Newton)

# common shared source
file(GLOB common_sources RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
    sources/ai/*
    sources/engine/*
    sources/generate/*
    sources/graphics/*
    sources/gui/*
    sources/haptic/*
    sources/input/*
    sources/math/*
    sources/physics/*
    sources/resources/*
    sources/scene/*
    sources/sound/*
    sources/system/*

    include/ai/*
    include/engine/*
    include/generate/*
    include/graphics/*
    include/gui/*
    include/haptic/*
    include/input/*
    include/math/*
    include/physics/*
    include/resources/*
    include/scene/*
    include/sound/*
    include/system/*
    include/*.*
)
list(APPEND common_sources include/BuildID_HPL2_0.h)
list(REMOVE_DUPLICATES common_sources)

#setup various IMPL sources
file(GLOB impl_sources RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
    include/impl/tinyXML/*
    include/impl/*.*

    # tinyXML
    sources/impl/tinyXml/*
    sources/impl/XmlDocumentTiny.cpp
    # scripting
    sources/impl/SqScript.cpp
    sources/impl/scriptarray.cpp
    sources/impl/scripthelper.cpp
    sources/impl/scriptstdstring.cpp
    sources/impl/scriptstdstring_utils.cpp
    # Image loading
    sources/impl/BitmapLoader*
    # theora video
    sources/impl/VideoStreamTheora.cpp
    # Newton
    sources/impl/*Newton.cpp
    # GL
    sources/impl/FrameBufferGL.cpp
    sources/impl/GLSL*
    sources/impl/OcclusionQueryOGL.cpp
    sources/impl/VertexBufferOGL_Array.cpp
    sources/impl/VertexBufferOGL_VBO.cpp
    sources/impl/VertexBufferOpenGL.cpp
    # SDL
    sources/impl/GamepadSDL.cpp
    sources/impl/GamepadSDL2.cpp
    sources/impl/KeyboardSDL.cpp
    sources/impl/MouseSDL.cpp
    sources/impl/MutexSDL.cpp
    sources/impl/ThreadSDL.cpp
    sources/impl/TimerSDL.cpp
    sources/impl/LowLevelGraphicsSDL.cpp
    sources/impl/LowLevelInputSDL.cpp
    sources/impl/LowLevelResourcesSDL.cpp
    sources/impl/LowLevelSystemSDL.cpp
    sources/impl/SDLEngineSetup.cpp
    sources/impl/SDLFontData.cpp
    sources/impl/SDLTexture.cpp
    # OpenAL
    sources/impl/LowLevelSoundOpenAL.cpp
    sources/impl/OpenAL*
    # mesh loader
    sources/impl/MeshLoaderCollada.cpp
    sources/impl/MeshLoaderColladaHelpers.cpp
    sources/impl/MeshLoaderColladaLoader.cpp
    sources/impl/MeshLoaderMSH.cpp
)

IF(APPLE)
    LIST(APPEND impl_sources
        sources/impl/PlatformMacOSX.mm
        sources/impl/PlatformUnix.cpp
        sources/impl/PlatformSDL.cpp
    )
ELSEIF(WIN32)
    LIST(APPEND impl_sources
        sources/impl/PlatformWin32.cpp
        sources/impl/MutexWin32.cpp
        sources/impl/ThreadWin32.cpp
        sources/impl/PlatformSDL.cpp
    )
ELSEIF(LINUX)
    LIST(APPEND impl_sources
        sources/impl/PlatformUnix.cpp
        sources/impl/PlatformSDL.cpp
    )
ENDIF()

# Setup buildID source

IF(APPLE)
    SET(VERSION_SUFFIX MacOSX)
ELSEIF(LINUX)
    SET(VERSION_SUFFIX Linux)
ENDIF()

SET(BUILDID_SUFFIX ${BUILDID_SUFFIX} CACHE STRING "Build ID Suffix")
find_file(BUILDID_BIN buildcounter.pl
    PATHS ${CMAKE_CURRENT_SOURCE_DIR}
    NO_DEFAULT_PATH
)
mark_as_advanced(BUILDID_SUFFIX BUILDID_BIN)

function(GenerateBuildID outvar idname includedir sourcedir)
    add_custom_command(
        OUTPUT
            ${sourcedir}/BuildID_${idname}_${BUILDID_SUFFIX}.cpp
            ${includedir}/BuildID_${idname}.h
        COMMAND
            ${BUILDID_BIN}
        ARGS
            ${idname} ${includedir} ${sourcedir}
    )
    set(${outvar} ${sourcedir}/BuildID_${idname}_${BUILDID_SUFFIX}.cpp
            ${includedir}/BuildID_${idname}.h PARENT_SCOPE)
endfunction()

GenerateBuildID(version_source
  HPL2_0
  ${CMAKE_CURRENT_SOURCE_DIR}/include
  ${CMAKE_CURRENT_SOURCE_DIR}/sources)

configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/include/ResourcesLocation.h.in
  ${CMAKE_CURRENT_SOURCE_DIR}/include/ResourcesLocation.h)

# setup source groups
source_group("Implementation Specific" REGULAR_EXPRESSION "\\/impl\\/.+")
source_group("TinyXML" REGULAR_EXPRESSION "\\/impl\\/tiny.+")
source_group("SDL Specific" REGULAR_EXPRESSION "\\/impl\\/.*SDL.+")
source_group("OpenGL Specific" REGULAR_EXPRESSION "\\/impl\\/.*GL.+")
source_group("HaptX Specific" REGULAR_EXPRESSION "\\/impl\\/.+HaptX.+")
source_group("Newton Specific" REGULAR_EXPRESSION "\\/impl\\/.+Newton.+")
source_group("Mesh Loaders" REGULAR_EXPRESSION "\\/impl\\/Mesh.+")
source_group("OpenAL Specific" REGULAR_EXPRESSION "\\/impl\\/.*OpenAL.+")
source_group("Win32" REGULAR_EXPRESSION "\\/impl\\/.+Win32.+")
source_group("Platform" REGULAR_EXPRESSION "\\/impl\\/Platform.+")


# setup libs
set(OpenGL_GL_PREFERENCE LEGACY)
find_package(OpenGL REQUIRED)
find_package(Theora REQUIRED)
find_package(Devil REQUIRED)
find_package(GLEW REQUIRED)
find_package(LibXml2 REQUIRED)

# setup HPL2 compile target
add_library(HPL2 STATIC
    ${common_sources}
    ${version_sources}
    ${impl_sources}
)

set(HPL2_INCLUDES
    PUBLIC include
    PRIVATE ${OALWRAPPER_INCLUDE_DIR}
    PRIVATE ${THEORA_INCLUDE_DIR}
    PRIVATE ${DEVIL_INCLUDE_DIR}
    PRIVATE ${GLEW_INCLUDE_DIRS}
    PRIVATE ${OPENGL_INCLUDE_DIR}
    PRIVATE ${LIBXML2_INCLUDE_DIR}
    PUBLIC ../dependencies/include
    PUBLIC ../dependencies/AngelScript/include
    PUBLIC ../dependencies/Newton/include
)

target_include_directories(HPL2
    ${HPL2_INCLUDES}
)

if(LINUX)
  set(PLATFORM_LIBS
    pthread
    dl
    )
endif()

target_link_libraries(HPL2
  OALWrapper
  AngelScript
  Newton
  ${GLEW_LIBRARIES}
  ${OPENGL_gl_LIBRARY}
  ${OPENGL_glu_LIBRARY}
  ${THEORA_LIBRARY}
  ${DEVIL_LIBRARY}
  ${PLATFORM_LIBS}
  z
  ${LIBXML2_LIBRARIES}
)
