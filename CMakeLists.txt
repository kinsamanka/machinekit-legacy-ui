cmake_minimum_required(VERSION 2.8)
project(Machinekit_Legacy_UI)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} 
	"${CMAKE_SOURCE_DIR}/cmake/Modules/")

#~ Required packages
find_package(TCL 8.5 REQUIRED)
find_package(Threads REQUIRED)
find_package(Curses REQUIRED)
find_package(X11 REQUIRED)
find_package(LibDL REQUIRED)
find_package(LinuxCNC REQUIRED)

find_package(PythonInterp 2.7 REQUIRED)
find_package(PythonLibs 2.7 REQUIRED)

if (PYTHON_VERSION_STRING AND PYTHONLIBS_VERSION_STRING)
	if (NOT PYTHON_VERSION_STRING VERSION_EQUAL 
			PYTHONLIBS_VERSION_STRING)
	message ( FATAL_ERROR
             "Version mismatch between python interpreter and libraries")
	endif (NOT PYTHON_VERSION_STRING VERSION_EQUAL 
			PYTHONLIBS_VERSION_STRING)
endif (PYTHON_VERSION_STRING AND PYTHONLIBS_VERSION_STRING)


find_package(PkgConfig)
pkg_check_modules(XAW xaw7)

set(Boost_DEBUG FALSE)
find_package(Boost 1.49.0 REQUIRED COMPONENTS thread serialization-mt)

include_directories(${LINUXCNC_INCLUDE_DIR})
set(MACHINEKIT_LIBRARIES ${LCNC_nml_lib} ${LCNC_linuxcncini_lib}
	${LCNC_linuxcnc_lib})

add_library(shcom STATIC shcom.cc shcom.hh)

add_library(linuxcnc MODULE shcom.cc emcsh.cc)
set_target_properties(linuxcnc PROPERTIES PREFIX "")
set_property(TARGET linuxcnc APPEND PROPERTY 
	INCLUDE_DIRECTORIES ${TCL_INCLUDE_PATH})
target_link_libraries(linuxcnc ${TCL_LIBRARY})

add_executable(linuxcncrsh emcrsh.cc)
target_link_libraries(linuxcncrsh shcom ${MACHINEKIT_LIBRARIES} 
	${CMAKE_THREAD_LIBS_INIT})

add_executable(schedrmt emcsched.cc emcsched.hh schedrmt.cc)
target_link_libraries(schedrmt shcom ${MACHINEKIT_LIBRARIES} 
	${CMAKE_THREAD_LIBS_INIT})

add_executable(linuxcnclcd sockets.c sockets.h emclcd.cc)
target_link_libraries(linuxcnclcd shcom ${MACHINEKIT_LIBRARIES})

add_executable(keystick keystick.cc)
target_link_libraries(keystick ${MACHINEKIT_LIBRARIES} ${CURSES_LIBRARIES})

add_executable(xlinuxcnc xemc.cc)
target_link_libraries(xlinuxcnc ${XAW_LIBRARIES} ${MACHINEKIT_LIBRARIES} 
	${X11_Xt_LIB} ${X11_X11_LIB} ${X11_SM_LIB} ${X11_ICE_LIB})

add_executable(halui halui.cc)
set_target_properties(halui PROPERTIES COMPILE_DEFINITIONS "ULAPI") 
target_link_libraries(halui ${LCNC_linuxcnchal_lib} ${MACHINEKIT_LIBRARIES})

add_executable(linuxcncweb 
	emccontroller.cc emccontroller.hh emcweb.cc 
	emcweb/mongoose.c emcweb/mongoose_wrapper.cc
	emcweb/mongoose.h emcweb/mongoose_wrapper.cc
	emcweb/singleton.hh)
set_target_properties(linuxcncweb 
	PROPERTIES COMPILE_DEFINITIONS "__DIR__=\"/var/cache/linuxcnc\"")
target_link_libraries(linuxcncweb shcom ${MACHINEKIT_LIBRARIES} 
	${CMAKE_THREAD_LIBS_INIT} ${LIBDL_LIBRARIES} ${Boost_LIBRARIES})

set(targets linuxcncrsh schedrmt linuxcnclcd keystick xlinuxcnc
	halui linuxcncweb linuxcnc)

# install targets
install (TARGETS ${targets} 
	DESTINATION bin
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib)

#~ This macro performs syntax checking on python files and copies 
#~ it to the destination dir. Renames the file depending on ${ext} parameter
macro(add_python_target tgt dest ext)
  foreach(file ${ARGN})
    set(OUT ${dest}/${file}${ext})
    list(APPEND OUT_FILES ${OUT})
    add_custom_command(OUTPUT ${OUT}
        COMMAND ${PYTHON_EXECUTABLE} -c 'import sys\;
		compile(open(sys.argv[1]).read(), sys.argv[1], \"exec\")' 
		${CMAKE_CURRENT_SOURCE_DIR}/${file}.py
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/${file}.py
			${OUT}
	DEPENDS ${file}.py)
  endforeach()
  add_custom_target(${tgt} ALL DEPENDS ${OUT_FILES})
endmacro()

#~ Create __init__.py
macro(add_python_init_target tgt)
  set(dest ${CMAKE_CURRENT_BINARY_DIR}/python/${PROJECT_NAME}/__init__.py)
  add_custom_command(OUTPUT ${dest}
    COMMAND ${CMAKE_COMMAND} -E touch ${dest})
  add_custom_target(${tgt} ALL DEPENDS ${dest})
endmacro()

#~ This macro copies glade files to destination  dir
macro(add_glade_target tgt)
  foreach(file ${ARGN})
    set(OUT ${CMAKE_CURRENT_BINARY_DIR}/${file})
    list(APPEND OUT_FILES ${OUT})
    add_custom_command(OUTPUT ${OUT}
	COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/${file}
			${OUT}
	DEPENDS ${file})
  endforeach()
  add_custom_target(${tgt} ALL DEPENDS ${OUT_FILES})
endmacro()

execute_process(COMMAND ${PYTHON_EXECUTABLE} -c
		"import site, sys; sys.stdout.write(site.PREFIXES[-1])"
	OUTPUT_VARIABLE PYTHON_PREFIX)
file(TO_CMAKE_PATH "${PYTHON_PREFIX}" PYTHON_PREFIX)

execute_process(COMMAND ${PYTHON_EXECUTABLE} -c
		"import site, sys; sys.stdout.write(site.getsitepackages()[-1])"
	OUTPUT_VARIABLE PYTHON_SITE_DIR)
file(TO_CMAKE_PATH "${PYTHON_SITE_DIR}" PYTHON_SITE_DIR)

string(REGEX REPLACE "^${PYTHON_PREFIX}/" ""
	PYTHON_SITE_DIR "${PYTHON_SITE_DIR}")

#~ include the python based programs
add_subdirectory(axis)
add_subdirectory(touchy)
add_subdirectory(stepconf)
add_subdirectory(pncconf)
add_subdirectory(gremlin)
add_subdirectory(gscreen)
add_subdirectory(gmoccapy)