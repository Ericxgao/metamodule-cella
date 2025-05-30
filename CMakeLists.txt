cmake_minimum_required(VERSION 3.24)

if(NOT "${METAMODULE_SDK_DIR}" STREQUAL "")
	message("METAMODULE_SDK_DIR set by CMake variable ${METAMODULE_SDK_DIR}")
elseif (DEFINED ENV{METAMODULE_SDK_DIR})
    set(METAMODULE_SDK_DIR "$ENV{METAMODULE_SDK_DIR}")
	message("METAMODULE_SDK_DIR set by environment variable ${METAMODULE_SDK_DIR}")
else()
    set(METAMODULE_SDK_DIR "${CMAKE_CURRENT_LIST_DIR}/../metamodule-plugin-sdk")
	message("METAMODULE_SDK_DIR set to default: ${METAMODULE_SDK_DIR}")
endif()
include(${METAMODULE_SDK_DIR}/plugin.cmake)

project(Cella VERSION 1.0.0 DESCRIPTION "Cella for Rack" LANGUAGES C CXX)

# Set C++17 standard
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Add option for virtual build
option(BUILD_VIRTUAL "Build for virtual environment" ON)

# Define source directories
set(SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/CellaVCV)

# Main library target
add_library(Cella STATIC)

# Compiler definitions
target_compile_definitions(Cella PRIVATE 
    __V1x 
    __PLUGIN
    $<$<BOOL:${BUILD_VIRTUAL}>:BUILD_VIRTUAL>
    _MB_CAPABLE=1
)

# Compiler options
target_compile_options(Cella PRIVATE
    -fpermissive
    -fdata-sections
    -ffunction-sections
    -fno-exceptions
    -fno-threadsafe-statics
    -fno-use-cxa-atexit
    $<$<CXX_COMPILER_ID:GNU>:-fvisibility=hidden>
    $<$<CXX_COMPILER_ID:GNU>:-fvisibility-inlines-hidden>
)

# Linker options
target_link_options(Cella PRIVATE
    -Wl,--gc-sections
)

# Source files
set(CELLA_SOURCES
    "${SOURCE_DIR}/src/plugin.cpp"
    "${SOURCE_DIR}/src/CognitiveShift.cpp"
    "${SOURCE_DIR}/src/Resonators.cpp"
    "${SOURCE_DIR}/src/Rich.cpp"
    "${SOURCE_DIR}/src/TwinPeaks.cpp"
    "${SOURCE_DIR}/src/Bezier.cpp"
    "${SOURCE_DIR}/src/Bytebeat.cpp"
    "${SOURCE_DIR}/src/Euler.cpp"
    "${SOURCE_DIR}/src/Integral.cpp"
    "${SOURCE_DIR}/src/2State.cpp"
)

file(GLOB_RECURSE FILTER_SOURCES
    "${SOURCE_DIR}/src/filter/*.cpp"
)

target_sources(Cella PRIVATE
    ${CELLA_SOURCES}
    ${FILTER_SOURCES}
)

# Include directories
target_include_directories(Cella PRIVATE 
    ${SOURCE_DIR}/src
    ${SOURCE_DIR}/src/filter
    ${METAMODULE_SDK_DIR}/metamodule-plugin-libc/newlib/libc/ctype
)

# Create the plugin
create_plugin(
    SOURCE_LIB      Cella
    PLUGIN_NAME     Cella
    PLUGIN_JSON     ${SOURCE_DIR}/plugin.json
    SOURCE_ASSETS   ${CMAKE_CURRENT_LIST_DIR}/assets
    DESTINATION     ${CMAKE_CURRENT_LIST_DIR}/metamodule-plugins
) 
