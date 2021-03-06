cmake_policy(SET CMP0074 NEW)

if(NOT realbits)
  set(realbits 64)
endif()


find_package(MPI COMPONENTS Fortran)

if(NOT MPI_Fortran_FOUND)
  message(STATUS "MUMPS skipped due to missing MPI")
  return()
endif()

# --- Metis
# Default is PORD instead of METIS.
# find_package(METIS)
# if(METIS_FOUND)
# add_executable(metis1 metis.f90)
# target_compile_options(metis1 PRIVATE ${FFLAGS})
# target_link_libraries(metis1 PRIVATE ${METIS_LIBRARIES})
# target_include_directories(metis1 PRIVATE ${METIS_INCLUDE_DIRS})

# add_test(NAME Metis
  # COMMAND ${MPIEXEC_EXECUTABLE} ${MPIEXEC_NUMPROC_FLAG} ${MPIEXEC_MAX_NUMPROCS} $<TARGET_FILE:metis1>
  # WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
# endif()

# --- Lapack
# Lapack must work correctly for Scalapack to work
find_package(LAPACK)
if(NOT LAPACK_FOUND)
  message(STATUS "MUMPS skipped due to missing Lapack")
  return()
endif()

# --- Scalapack
# Scalapack must work correctly for MUMPS to work
find_package(SCALAPACK)
if(NOT SCALAPACK_FOUND)
  message(STATUS "MUMPS skipped due to missing Scalapack")
  return()
endif()


add_executable(blacs1 blacs.f)
target_compile_options(blacs1 PRIVATE ${FFLAGS})
target_link_libraries(blacs1 PRIVATE ${SCALAPACK_LIBRARIES} ${LAPACK_LIBRARIES} MPI::MPI_Fortran)
target_include_directories(blacs1 PRIVATE ${SCALAPACK_INCLUDE_DIRS})
add_test(NAME BLACS COMMAND ${MPIEXEC_EXECUTABLE} ${MPIEXEC_NUMPROC_FLAG} ${MPIEXEC_MAX_NUMPROCS} $<TARGET_FILE:blacs1>)
set_tests_properties(BLACS PROPERTIES RUN_SERIAL true TIMEOUT 15)

add_executable(scalapack1 scalapack_example1.f)
target_compile_options(scalapack1 PRIVATE ${FFLAGS})
target_link_libraries(scalapack1 PRIVATE ${SCALAPACK_LIBRARIES} ${LAPACK_LIBRARIES} MPI::MPI_Fortran)
# note: this test needs at least 6 MPI processes, even if you're on a very small computer it's OK.
add_test(NAME Scalapack
  COMMAND ${MPIEXEC_EXECUTABLE} ${MPIEXEC_NUMPROC_FLAG} 6 --oversubscribe $<TARGET_FILE:scalapack1>
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
set_tests_properties(Scalapack PROPERTIES RUN_SERIAL true TIMEOUT 15)

# --- MUMPS

include(mumps.cmake)

if(NOT MUMPS_FOUND)
  message(STATUS "MUMPS skipped")
  return()
endif()

add_executable(testmumps test_mumps.f90)
target_compile_options(testmumps PRIVATE ${FFLAGS})
target_include_directories(testmumps PRIVATE ${MUMPS_INCLUDE_DIRS})
target_link_libraries(testmumps PRIVATE
                     ${MUMPS_LIBRARIES}
                     MPI::MPI_Fortran)

add_test(NAME MUMPS
  COMMAND ${MPIEXEC_EXECUTABLE} ${MPIEXEC_NUMPROC_FLAG} ${MPIEXEC_MAX_NUMPROCS} $<TARGET_FILE:testmumps>
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
set_tests_properties(MUMPS PROPERTIES RUN_SERIAL true TIMEOUT 15)

  # manual:
# mpif90 -m64 ../test_mumps.f90 -I$HOME/flibs-gnu-mkl/MUMPS/include/ -L$HOME/flibs-gnu-mkl/MUMPS/lib -ldmumps -lmumps_common -lpord -L${MKLROOT}/lib/intel64 -Wl,--no-as-needed -lmkl_scalapack_lp64 -lmkl_intel_lp64 -lmkl_sequential -lmkl_core -lmkl_blacs_openmpi_lp64 -lpthread -lm -ldl -L$HOME/flibs-gnu-mkl/scotch/libs -lscotch -lesmumps -L$HOME/flibs-gnu-mkl/metis/build/Linux-x86_64/libmetis/ -lmetis

# CMake
# /usr/bin/f95    -m64 -I${MKLROOT}/include -I$HOME/flibs-gnu-mkl/MUMPS/include -I/usr/lib/x86_64-linux-gnu/openmpi/include -I/usr/lib/x86_64-linux-gnu/openmpi/lib ../test_mumps.f90  -o testmumps -Wl,-rpath,/usr/lib/x86_64-linux-gnu/openmpi/lib /home/thin/flibs-gnu-mkl/MUMPS/lib/libdmumps.a /home/thin/flibs-gnu-mkl/MUMPS/lib/libmumps_common.a /home/thin/flibs-gnu-mkl/MUMPS/lib/libpord.a -lmkl_scalapack_lp64 -lmkl_intel_lp64 -lmkl_sequential -lmkl_core -lmkl_blacs_intelmpi_lp64 -lpthread -ldl -lm /home/thin/flibs-gnu-mkl/scotch/lib/libesmumps.a /home/thin/flibs-gnu-mkl/scotch/lib/libscotch.a /home/thin/flibs-gnu-mkl/scotch/lib/libscotcherrexit.a /home/thin/flibs-gnu-mkl/metis/build/Linux-x86_64/libmetis/libmetis.a /usr/lib/x86_64-linux-gnu/libopenblas.so -lmkl_intel_lp64 -lmkl_intel_thread -lmkl_core /usr/lib/x86_64-linux-gnu/libiomp5.so -L/usr/lib -pthread /usr/lib/x86_64-linux-gnu/openmpi/lib/libmpi_usempif08.so /usr/lib/x86_64-linux-gnu/openmpi/lib/libmpi_usempi_ignore_tkr.so /usr/lib/x86_64-linux-gnu/openmpi/lib/libmpi_mpifh.so /usr/lib/x86_64-linux-gnu/openmpi/lib/libmpi.so
