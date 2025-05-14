# Initialize include directories and library variables
set(BTRACK_INCLUDE_DIRS "")
set(BTRACK_LIBRARIES "")
set(BTRACK_LIBRARY_DIRS "")

# Add base include directories
list(APPEND BTRACK_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/../src)
set(KISS_FFT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../libs/kiss_fft130)
list(APPEND BTRACK_INCLUDE_DIRS ${KISS_FFT_DIR})
set(BTRACK_LIBRARIES "")

IF (WIN32)
    # Find libsamplerate
    message(STATUS "Windows System...")
    
	set(WIN_INCLUDE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../libs/libsamplerate-0.2.2-win64/include")
    set(SAMPLERATE_VAR "${CMAKE_CURRENT_SOURCE_DIR}/../libs/libsamplerate-0.2.2-win64/lib")
	list(APPEND BTRACK_INCLUDE_DIRS ${WIN_INCLUDE_DIR})    
    list(APPEND BTRACK_LIBRARY_DIRS ${SAMPLERATE_VAR})
    message(STATUS "${SAMPLERATE_VAR}")
    find_library(LIBSAMPLERATE_LIBRARIES NAMES samplerate PATHS ${SAMPLERATE_VAR})
    list(APPEND BTRACK_LIBRARIES ${LIBSAMPLERATE_LIBRARIES})

ELSEIF(UNIX)
    IF(APPLE)
        message(STATUS "Mac System...")
        # Add both Homebrew paths for Intel and Apple Silicon Macs
        set(MAC_INCLUDE_DIRS "/opt/homebrew/include" "/usr/local/include")
        set(MAC_LIB_DIRS "/opt/homebrew/lib" "/usr/local/lib")
		list(APPEND BTRACK_INCLUDE_DIRS ${MAC_INCLUDE_DIRS})
        list(APPEND BTRACK_LIBRARY_DIRS ${MAC_LIB_DIRS})
		
        # # For immediate linking in this build
        # link_directories(${MAC_LIB_DIRS})
        
        # Find library with explicit path hints
        find_library(LIBSAMPLERATE_LIBRARIES NAMES samplerate 
                    PATHS ${MAC_LIB_DIRS}
                    NO_DEFAULT_PATH)
        list(APPEND BTRACK_LIBRARIES ${LIBSAMPLERATE_LIBRARIES})
        
        # Also find include path for samplerate.h
        find_path(LIBSAMPLERATE_DIR NAMES samplerate.h
                 PATHS ${MAC_INCLUDE_DIRS}
                 NO_DEFAULT_PATH)
        if(LIBSAMPLERATE_DIR)
            list(APPEND BTRACK_INCLUDE_DIRS ${LIBSAMPLERATE_DIR})
        endif()
    ELSE()
        # It means we are in a UNIX system (most likely)
        message(STATUS "Linux System...")
        find_library(LIBSAMPLERATE_LIBRARIES NAMES samplerate)
        list(APPEND BTRACK_LIBRARIES ${LIBSAMPLERATE_LIBRARIES})
        
        find_path(LIBSAMPLERATE_DIR NAMES samplerate.h)
        if(LIBSAMPLERATE_DIR)
            list(APPEND BTRACK_INCLUDE_DIRS ${LIBSAMPLERATE_DIR})
        endif()
    ENDIF()
ENDIF()

# Print the found library to help with debugging
if(LIBSAMPLERATE_LIBRARIES)
    message(STATUS "Found libsamplerate: ${LIBSAMPLERATE_LIBRARIES}")
else()
    message(FATAL_ERROR "Could not find libsamplerate library")
endif()

add_library (BTrack STATIC
    BTrack.cpp
    BTrack.h
    OnsetDetectionFunction.cpp
    OnsetDetectionFunction.h
    CircularBuffer.h
    ${KISS_FFT_DIR}/kiss_fft.c
)

source_group (Source src)

target_compile_definitions (BTrack PUBLIC -DUSE_KISS_FFT)

# Propagate paths to parent scope
target_include_directories(BTrack PUBLIC ${BTRACK_INCLUDE_DIRS})
target_link_libraries(BTrack PUBLIC ${BTRACK_LIBRARIES})

# Export variables to parent scope
# set(BTRACK_INCLUDE_DIRS ${BTRACK_INCLUDE_DIRS} CACHE INTERNAL "BTrack include directories")
# set(BTRACK_LIBRARIES ${BTRACK_LIBRARIES} CACHE INTERNAL "BTrack dependent libraries")
# set(BTRACK_LIBRARY_DIRS ${BTRACK_LIBRARY_DIRS} CACHE INTERNAL "BTrack library directories")