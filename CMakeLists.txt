cmake_minimum_required (VERSION 3.10)
project (Quiet C)

set(QUIET_PROFILES_LOCATION "${CMAKE_INSTALL_PREFIX}/share/quiet/")
add_definitions(-DQUIET_PROFILES_LOCATION="${QUIET_PROFILES_LOCATION}quiet-profiles.json")

if (CMAKE_C_COMPILER_ID MATCHES "GNU" OR CMAKE_C_COMPILER_ID MATCHES "Clang")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O2 -g -Wpedantic -Wall -D_XOPEN_SOURCE=700 -std=c99")
endif()

# Dependencies
# liquid and jansson should be available as targets
set(CORE_DEPENDENCIES liquid-static jansson m)

set(CMAKE_THREAD_PREFER_PTHREAD ON)
find_package(Threads)
if (Threads_FOUND)
  add_definitions(-DRING_BLOCKING=1)
  add_definitions(-DQUIET_PTHREAD_ERROR=1)
  set(SRCFILES src/demodulator.c src/modulator.c src/utility.c src/decoder.c src/encoder.c src/profile.c src/error.c src/ring_blocking.c)
  list(APPEND CORE_DEPENDENCIES Threads::Threads)
else()
  add_definitions(-DRING_BLOCKING=0)
  add_definitions(-DQUIET_PTHREAD_ERROR=0)
  set(SRCFILES src/demodulator.c src/modulator.c src/utility.c src/decoder.c src/encoder.c src/profile.c src/error.c src/ring.c)
endif()

add_library(quiet_static STATIC ${SRCFILES})
target_link_libraries(quiet_static PUBLIC ${CORE_DEPENDENCIES})
set_target_properties(quiet_static PROPERTIES OUTPUT_NAME "quiet")

# Interface for consumers
target_include_directories(quiet_static PUBLIC 
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
)

# Alias for compatibility
if(NOT TARGET quiet)
    add_library(quiet ALIAS quiet_static)
endif()

# Copy quiet.h to binary dir if needed by some weird legacy stuff
add_custom_command(TARGET quiet_static POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_BINARY_DIR}/include
    COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/include/quiet.h ${CMAKE_CURRENT_BINARY_DIR}/include/quiet.h
)
