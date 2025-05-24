# Initialize include directories and library variables in an old fashioned way!
set(BTRACK_INCLUDE_DIRS "")
set(BTRACK_LIBRARIES "")

# Add base include directories
list(APPEND BTRACK_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/../src)

# kiss_fft
set(KISS_FFT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../libs/kiss_fft130)
list(APPEND BTRACK_INCLUDE_DIRS ${KISS_FFT_DIR})
add_library(kissfft STATIC ${KISS_FFT_DIR}/kiss_fft.c)
target_include_directories(kissfft PUBLIC ${KISS_FFT_DIR})
list(APPEND BTRACK_LIBRARIES kissfft)

# libsamplerate
set(LIBSAMPLERATE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../libs/libsamplerate)

# Prevent missing submodule
if(NOT EXISTS "${LIBSAMPLERATE_DIR}/CMakeLists.txt")
    message(FATAL_ERROR "libsamplerate submodule not found. Please run 'git submodule update --init --recursive'")
endif()

add_subdirectory(${LIBSAMPLERATE_DIR} libsamplerate_build)

list(APPEND BTRACK_LIBRARIES samplerate)
list(APPEND BTRACK_INCLUDE_DIRS ${LIBSAMPLERATE_DIR}/include)

# Create BTrack library
add_library (BTrack STATIC
    BTrack.cpp
    BTrack.h
    OnsetDetectionFunction.cpp
    OnsetDetectionFunction.h
    CircularBuffer.h
)

target_compile_definitions (BTrack PUBLIC -DUSE_KISS_FFT)
target_include_directories(BTrack PUBLIC ${BTRACK_INCLUDE_DIRS})
target_link_libraries(BTrack PUBLIC ${BTRACK_LIBRARIES})