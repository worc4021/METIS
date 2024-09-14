cmake_minimum_required(VERSION 3.5)
project(METIS C)

include(GNUInstallDirs)

include($ENV{CMAKE_INSTALL_PREFIX}/cmake/GKlibTargets.cmake)

include(./conf/gkbuild.cmake)

# METIS' custom options
option(IDX64 "enable 64 bit ints" OFF)
option(REAL64 "enable 64 bit floats (i.e., double)" OFF)
if(IDX64)
 set(METIS_COPTIONS "${METIS_COPTIONS} -DIDXTYPEWIDTH=64")
else()
 set(METIS_COPTIONS "${METIS_COPTIONS} -DIDXTYPEWIDTH=32")
endif(IDX64)
if(REAL64)
 set(METIS_COPTIONS "${METIS_COPTIONS} -DREALTYPEWIDTH=64")
else()
 set(METIS_COPTIONS "${METIS_COPTIONS} -DREALTYPEWIDTH=32")
endif(REAL64)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${METIS_COPTIONS}")


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

