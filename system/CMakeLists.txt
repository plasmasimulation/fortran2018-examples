
add_executable(color color_text.f90)
target_compile_options(color PRIVATE ${FFLAGS})
target_link_libraries(color ${FLIBS})
add_test(NAME Color COMMAND color)

add_library(osdet os_detect.f90)
target_compile_options(osdet PRIVATE ${FFLAGS})
target_link_libraries(osdet ${FLIBS})

if(NOT f08command)
  return()
endif()

add_executable(gitrev gitrev.f90)
target_compile_options(gitrev PRIVATE ${FFLAGS})
target_link_libraries(gitrev ${FLIBS})
add_test(NAME Git COMMAND gitrev)

add_executable(complog compiler_log.f90)
target_compile_options(complog PRIVATE ${FFLAGS})
target_link_libraries(complog ${FLIBS})
add_test(NAME Compiler COMMAND complog)

find_package(FFMPEG COMPONENTS PLAY)

add_executable(playsound play_sound.f90)
target_compile_options(playsound PRIVATE ${FFLAGS})
target_link_libraries(playsound ${FLIBS})
if(FFMPEG_FOUND)
  add_test(NAME PlaySound COMMAND $<TARGET_FILE:playsound> ${CMAKE_CURRENT_SOURCE_DIR}/bell.aac)
  set_tests_properties(PlaySound PROPERTIES TIMEOUT 15)
endif()

if(f18errorstop)
if(WIN32 AND CMAKE_Fortran_COMPILER_ID STREQUAL PGI)
  message(STATUS "PGI Windows bug")
else()
  add_executable(callpython call_python_script.f90)
  target_compile_options(callpython PRIVATE ${FFLAGS})
  target_link_libraries(callpython PRIVATE osdet ${FLIBS})
endif()
endif(f18errorstop)
