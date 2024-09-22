# These programs use internal metis data structures.
include_directories(../libmetis)

# Build program.
add_executable(gpmetis gpmetis.c cmdline_gpmetis.c io.c stat.c)
add_executable(ndmetis ndmetis.c cmdline_ndmetis.c io.c smbfactor.c)
add_executable(mpmetis mpmetis.c cmdline_mpmetis.c io.c stat.c)
add_executable(m2gmetis m2gmetis.c cmdline_m2gmetis.c io.c)
add_executable(graphchk graphchk.c io.c)
add_executable(cmpfillin cmpfillin.c io.c smbfactor.c)

# Link with the required libraries
foreach(prog gpmetis ndmetis mpmetis m2gmetis graphchk cmpfillin)
  target_link_libraries(${prog} metis GKlib::GKlib $<$<BOOL:${UNIX}>:m>)
endforeach(prog)


install(TARGETS gpmetis ndmetis mpmetis m2gmetis graphchk cmpfillin
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

add_test(NAME m2gmetis COMMAND m2gmetis ${PROJECT_SOURCE_DIR}/graphs/metis.mesh test.graph)
add_test(NAME graphchk COMMAND graphchk ${PROJECT_SOURCE_DIR}/graphs/4elt.graph)