cmake_minimum_required(VERSION 3.5)
project(METIS C)

include(GNUInstallDirs)

include($ENV{CMAKE_INSTALL_PREFIX}/cmake/GKlibTargets.cmake)

include(./conf/gkbuild.cmake)

# METIS' custom options
option(IDX64 "enable 64 bit ints" ON) # required for Mumps
option(REAL64 "enable 64 bit floats (i.e., double)" TRUE)

# Find sources.
file(GLOB metis_sources libmetis/*.c)

# Build libmetis.
add_library(metis ${metis_sources})

target_include_directories(metis PUBLIC 
                                $<BUILD_INTERFACE:libmetis include>
                                $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/metis>)
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
      RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}/${CMAKE_BUILD_TYPE})
install(FILES include/metis.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/metis)
install(EXPORT metisTargets
      FILE metisTargets.cmake
      NAMESPACE metis::
      DESTINATION cmake)

