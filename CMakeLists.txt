cmake_minimum_required (VERSION 3.12)
project (BTrack VERSION 1.0.4)

set (CMAKE_CXX_STANDARD 11)

if(APPLE)
    find_path(LIBSAMPLERATE_DIR NAMES samplerate.h)
    include_directories(${LIBSAMPLERATE_DIR})
endif()

option (BUILD_TESTS "Build tests" OFF)

add_subdirectory (src)

if (BUILD_TESTS)
    enable_testing()
    add_subdirectory (tests)
endif (BUILD_TESTS)

set (CMAKE_SUPPRESS_REGENERATION true)

