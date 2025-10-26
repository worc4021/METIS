cmake_minimum_required(VERSION 3.25)
project(METIS C)

include(GNUInstallDirs)

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

set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
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

# Find sources.
file(GLOB metis_sources ${CMAKE_CURRENT_SOURCE_DIR}/libmetis/*.c)
file(GLOB metis_headers ${CMAKE_CURRENT_SOURCE_DIR}/libmetis/*.h)

# Build libmetis.
add_library(metis ${METIS_LIBRARY_TYPE} ${metis_sources})
target_link_libraries(metis PUBLIC GKlib)
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

enable_testing()

add_subdirectory(programs)

install(TARGETS metis
      EXPORT metisTargets
      LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/${CMAKE_BUILD_TYPE}
      ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}/${CMAKE_BUILD_TYPE}
      RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}/${CMAKE_BUILD_TYPE}
      FILE_SET public_header_set)
install(EXPORT metisTargets
      FILE metisTargets.cmake
      NAMESPACE metis::
      DESTINATION cmake)

