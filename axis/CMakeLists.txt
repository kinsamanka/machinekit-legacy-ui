cmake_minimum_required(VERSION 2.8 )
project(axis)

include_directories(${PYTHON_INCLUDE_DIRS})
include_directories(${TCL_INCLUDE_PATH})

add_library(py_linuxcnc MODULE extensions/emcmodule.cc)
set_target_properties(py_linuxcnc PROPERTIES OUTPUT_NAME linuxcnc PREFIX "")

add_library(minigl MODULE extensions/minigl.c)
set_target_properties(minigl PROPERTIES PREFIX "")

add_library(_toglmodule MODULE extensions/_toglmodule.c 
	extensions/togl.h)
set_target_properties(_toglmodule PROPERTIES PREFIX "")

execute_process(COMMAND ${CMAKE_COMMAND} 
	-E make_directory ${CMAKE_CURRENT_BINARY_DIR}/scripts)

set(PROGRAMS scripts/axis 
	scripts/axis-remote
	scripts/linuxcnctop
	scripts/hal_manualtoolchange
	scripts/mdi
	scripts/image-to-gcode
	scripts/lintini
	scripts/debuglevel
	scripts/teach-in
	scripts/tracking-test)

add_python_target(axis ${CMAKE_CURRENT_BINARY_DIR} "" ${PROGRAMS})
	
foreach(PROG ${PROGRAMS})
	install (PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/${PROG}
		 DESTINATION bin)
endforeach(PROG)

install(FILES 
	${CMAKE_CURRENT_SOURCE_DIR}/etc/axis_big_dro
	${CMAKE_CURRENT_SOURCE_DIR}/etc/axis_light_background
	DESTINATION ${CMAKE_INSTALL_PREFIX}/share/doc/linuxcnc)
