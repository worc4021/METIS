cmake_minimum_required(VERSION 3.5)
project(METIS C)

include(GNUInstallDirs)

include($ENV{CMAKE_INSTALL_PREFIX}/cmake/GKlibTargets.cmake)

include(./conf/gkbuild.cmake)

# METIS' custom options
option(IDX64 "enable 64 bit ints" OFF)
option(REAL64 "enable 64 bit floats (i.e., double)" OFF)

# Recursively look for CMakeLists.txt in subdirs.
# add_subdirectory("libmetis")

# Find sources.
file(GLOB metis_sources libmetis/*.c)

# Build libmetis.
add_library(metis ${metis_sources})

target_include_directories(metis PUBLIC 
                                $<BUILD_INTERFACE:libmetis include>
                                $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/metis>)
target_link_libraries(metis GKlib::GKlib)
set(INDEXTYPEWIDTH "$<IF:$<BOOL:${IDX64}>,64,32>")
set(REALTYPEWIDTH "$<IF:$<BOOL:${REAL64}>,64,32>")
target_compile_definitions(metis PUBLIC IDXTYPEWIDTH=${INDEXTYPEWIDTH} REALTYPEWIDTH=${REALTYPEWIDTH})

add_subdirectory(programs)

install(TARGETS metis
      EXPORT metisTargets
      LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
      ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
      RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})
install(FILES include/metis.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/metis)
install(EXPORT metisTargets
      FILE metisTargets.cmake
      NAMESPACE metis::
      DESTINATION cmake)

