### tests
if (PHREEQCRM_FORTRAN_TESTING)
  project(FortranAdvect Fortran)
  
  # source
  SET(FortranAdvect_SOURCES
    advection_f90.F90
    ../src/RM_interface.F90
    ../src/IPhreeqcPhast/IPhreeqc/IPhreeqc_interface.F90
    species_f90.F90
  )
  
  # place [*.F[90]] files into "Source Files"
  source_group("Source Files" FILES "advection_f90.F90")
  source_group("Source Files" FILES "../src/RM_interface.F90")
  source_group("Source Files" FILES "../src/IPhreeqcPhast/IPhreeqc/IPhreeqc_interface.F90")
  source_group("Source Files" FILES "species_f90.F90")
  
  add_library(FortranAdvect STATIC ${FortranAdvect_SOURCES})
  
  set_target_properties(FortranAdvect PROPERTIES COMPILE_DEFINITIONS "TEST_FORTRAN")
endif()

project(TestRM CXX)

# Tests/advect.pqi
configure_file (
  "advect.pqi.in"
  "advect.pqi"
  )

# Tests/phreeqc.dat
configure_file (
  "phreeqc.dat.in"
  "phreeqc.dat"
  )
 
# Tests/units.pqi
configure_file (
  "units.pqi.in"
  "units.pqi"
  )

# source
SET(TestRM_SOURCES
  advection_c.c
  advection_cpp.cpp
  main.cpp
  species_c.c
  species_cpp.cpp
)

# test executable
add_executable(TestRM ${TestRM_SOURCES})

if (PHREEQCRM_FORTRAN_TESTING)
  set_target_properties(TestRM PROPERTIES COMPILE_DEFINITIONS "TEST_FORTRAN")
endif ()

# library dependencies
##{{
##if (ZLIB_FOUND)
##  set(EXTRA_LIBS ${EXTRA_LIBS} ${ZLIB_LIBRARIES})
##endif()  
##}}
if (PHREEQCRM_FORTRAN_TESTING)
  # need extra mpi fortran libs if using msmpi
  if (MPI_Fortran_LIBRARIES MATCHES "msmpi.lib$")
    STRING(REGEX REPLACE "msmpi.lib$" "msmpifec.lib" EXTRA_FORTRAN_LIB ${MPI_Fortran_LIBRARIES})
    SET(EXTRA_LIBS ${EXTRA_LIBS} ${EXTRA_FORTRAN_LIB})
  endif ()  
  if (PHREEQCRM_BUILD_MPI)
    SET(EXTRA_LIBS ${EXTRA_LIBS} FortranAdvect PhreeqcRM ${MPI_CXX_LIBRARIES})
  else ()
    SET(EXTRA_LIBS ${EXTRA_LIBS} FortranAdvect PhreeqcRM)
  endif ()
  # Override bug reported on:
  # http://www.cmake.org/pipermail/cmake/2009-July/030954.html
  if(WIN32 AND ${CMAKE_Fortran_COMPILER_ID} STREQUAL "Intel")  
    if (NOT CMAKE_Fortran_FLAGS_DEBUG MATCHES "/Od")
      set(CMAKE_Fortran_FLAGS_DEBUG "/Od ${CMAKE_Fortran_FLAGS_DEBUG}")
    endif()
  endif()
else ()
  if (PHREEQCRM_BUILD_MPI)
    SET(EXTRA_LIBS ${EXTRA_LIBS} PhreeqcRM ${MPI_CXX_LIBRARIES})
  else ()
    SET(EXTRA_LIBS ${EXTRA_LIBS} PhreeqcRM)
  endif ()
endif ()

# link (zlib must be last on linux)
if (PHREEQCRM_USE_ZLIB AND ZLIB_FOUND)
  set(EXTRA_LIBS ${EXTRA_LIBS} ${ZLIB_LIBRARIES})
endif()
target_link_libraries(TestRM ${EXTRA_LIBS})

# test compile and run
add_test(TestCompileAndRun TestRM)

if (MSVC AND BUILD_SHARED_LIBS)
  # copy dll
  add_custom_command(TARGET TestRM POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:PhreeqcRM> $<TARGET_FILE_DIR:TestRM>
  )
endif()


# MPI test (TODO Need to check if this works in Cmake 2.8)
# 2.8 => change $<TARGET_FILE:TestRM>
# get_property(exe_location TARGET testrm PROPERTY LOCATION)
# message (STATUS "exe_location = ${exe_location}")
if (PHREEQCRM_BUILD_MPI)
  add_test(NAME TestMPI
           COMMAND ${MPIEXEC} ${MPIEXEC_NUMPROC_FLAG} 3
           ${MPIEXEC_PREFLAGS}
           $<TARGET_FILE:TestRM>
           ${MPIEXEC_POSTFLAGS}
           )
endif ()
