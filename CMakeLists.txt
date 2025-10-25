cmake_minimum_required(VERSION 3.25)
project(METIS C)

set(SHARED FALSE CACHE BOOL "build a shared library")

if(MSVC)
  set(METIS_INSTALL FALSE)
else()
  set(METIS_INSTALL TRUE)
endif()

# Configure libmetis library.
if(SHARED)
  set(METIS_LIBRARY_TYPE SHARED)
else()
  set(METIS_LIBRARY_TYPE STATIC)
endif(SHARED)

include(FetchContent)
FetchContent_Declare(
  gklib
  GIT_REPOSITORY https://github.com/worc4021/GKlib.git
  GIT_TAG        master
  EXCLUDE_FROM_ALL
)
FetchContent_MakeAvailable(gklib)


# METIS' custom options
option(IDX64 "enable 64 bit ints" OFF)
option(REAL64 "enable 64 bit floats (i.e., double)" OFF)
set(METIS_COPTIONS "")
if(IDX64)
 list(APPEND METIS_COPTIONS "IDXTYPEWIDTH=64")
else()
 list(APPEND METIS_COPTIONS "IDXTYPEWIDTH=32")
endif(IDX64)
if(REAL64)
 list(APPEND METIS_COPTIONS "REALTYPEWIDTH=64")
else()
 list(APPEND METIS_COPTIONS "REALTYPEWIDTH=32")
endif(REAL64)

set(INSTALL_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)

add_subdirectory("libmetis")
add_subdirectory("programs")
