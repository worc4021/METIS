cmake_minimum_required(VERSION 3.25)
project(METIS C)

include(GNUInstallDirs)

include($ENV{CMAKE_INSTALL_PREFIX}/cmake/GKlibTargets.cmake)

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

target_link_libraries(metis GKlib::GKlib)
set_target_properties(metis PROPERTIES 
      POSITION_INDEPENDENT_CODE ON
      MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

if (IDX64)
      set(INDEXTYPEWIDTH 64)
else(IDX64)
      set(INDEXTYPEWIDTH 32)
endif(IDX64)

if (REAL64)
      set(REALTYPEWIDTH 64)
else(REAL64)
      set(REALTYPEWIDTH 32)
endif(REAL64)

target_compile_definitions(metis PUBLIC IDXTYPEWIDTH=${INDEXTYPEWIDTH} REALTYPEWIDTH=${REALTYPEWIDTH})
message(STATUS "Configured with IDXTYPEWIDTH=${INDEXTYPEWIDTH} REALTYPEWIDTH=${REALTYPEWIDTH}")

enable_testing()

add_subdirectory(programs)

install(TARGETS metis
      EXPORT metisTargets
      LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/${CMAKE_BUILD_TYPE}
      ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}/${CMAKE_BUILD_TYPE}
      RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}/${CMAKE_BUILD_TYPE}
      FILE_SET HEADERS)
install(EXPORT metisTargets
      FILE metisTargets.cmake
      NAMESPACE metis::
      DESTINATION cmake)

