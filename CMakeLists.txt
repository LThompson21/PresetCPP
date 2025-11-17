cmake_minimum_required(VERSION 3.21)

project(proj
  VERSION 0.1
  LANGUAGES CXX
)

# Always build with PIC so static->shared flips are painless
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# -------- Library --------
# Force a shared lib regardless of BUILD_SHARED_LIBS
add_executable(proj 
    "src/main.cpp"
)

# Public / Private include paths
# NOTE: generated goes PUBLIC because a public header may include it
target_include_directories(proj
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/>
)

# C++ standard
target_compile_features(proj PUBLIC cxx_std_23)

# Visibility (optional but nice on GCC/Clang)
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang|GNU")
  target_compile_options(proj PRIVATE -fvisibility=hidden)
endif()

if (WIN32)
    target_compile_definitions(proj PRIVATE NOMINMAX)
endif()

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
    target_compile_options(proj PRIVATE -Wno-deprecated-volatile)
endif()

# -------- Output dirs (nice for local dev) --------
set_target_properties(proj PROPERTIES
  RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
  LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
  ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
)

# -------- Build-tree package export --------
include(CMakePackageConfigHelpers)

set(PROJ_EXPORT_TARGETS proj)

# Export the targets from the *build tree*
export(
  TARGETS ${PROJ_EXPORT_TARGETS}
  NAMESPACE proj::
  FILE "${CMAKE_CURRENT_BINARY_DIR}/projTargets.cmake"
)

# Minimal config file for the build tree
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/projConfig.cmake" [=[
# Build-tree package config for proj
include("${CMAKE_CURRENT_LIST_DIR}/projTargets.cmake")
]=])
