cmake_minimum_required(VERSION 3.25)
project(METIS C)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# Configure libmetis library.
if(SHARED)
  set(METIS_LIBRARY_TYPE SHARED)
else()
  set(METIS_LIBRARY_TYPE STATIC)
endif(SHARED)

find_package(GKlib REQUIRED)


set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
# METIS' custom options
option(METIS_BUILD_TESTS "Build the METIS tests" ON)
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

# Find sources.
file(GLOB metis_sources ${CMAKE_CURRENT_SOURCE_DIR}/libmetis/*.c)
file(GLOB metis_headers ${CMAKE_CURRENT_SOURCE_DIR}/libmetis/*.h)
set(METIS_BASE_DIR ${CMAKE_CURRENT_SOURCE_DIR})
# Build libmetis.
add_library(metis ${METIS_LIBRARY_TYPE} ${metis_sources})
target_link_libraries(metis PUBLIC GKlib::GKlib)
target_sources(metis
      PRIVATE 
        FILE_SET private_header_set
        TYPE HEADERS 
        BASE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/libmetis
        FILES ${metis_headers}
)
target_sources(metis
      PUBLIC 
        FILE_SET public_header_set
        TYPE HEADERS
        BASE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/include
        FILES ${CMAKE_CURRENT_SOURCE_DIR}/include/metis.h
      )     
target_compile_definitions(metis PUBLIC ${METIS_COPTIONS})

if(METIS_BUILD_TESTS)
  include(CTest)
  enable_testing()
  add_subdirectory(programs)
endif()

configure_package_config_file(METISConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/METISConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/METIS)

install(TARGETS metis
      EXPORT METISTargets
      LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
      ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
      RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
      FILE_SET public_header_set)
install(EXPORT METISTargets
      FILE METISTargets.cmake
      NAMESPACE METIS::
      DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/METIS)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/METISConfig.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/METIS )
