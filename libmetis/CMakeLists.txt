# Find sources.
file(GLOB metis_sources *.c)

# Build libmetis.
add_library(metis ${METIS_LIBRARY_TYPE} ${metis_sources})
target_link_libraries(metis PUBLIC GKlib)
target_include_directories(metis PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
target_include_directories(metis PUBLIC ${INSTALL_INCLUDE_DIR})
target_compile_definitions(metis PUBLIC ${METIS_COPTIONS})

if(METIS_INSTALL)
  install(TARGETS metis
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION lib
    ARCHIVE DESTINATION lib)
endif()
