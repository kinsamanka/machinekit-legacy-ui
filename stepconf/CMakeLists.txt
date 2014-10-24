cmake_minimum_required(VERSION 2.8 )
project(stepconf)

set(PY_FILES pages
	build_INI
	build_HAL)

set(GLADE_FILES main_page.glade
	base.glade
	start.glade
	pport1.glade
	pport2.glade
	spindle.glade
	options.glade
	axisx.glade
	axisy.glade
	axisz.glade
	axisa.glade
	finished.glade)

execute_process(COMMAND ${CMAKE_COMMAND} 
	-E make_directory ${CMAKE_CURRENT_BINARY_DIR}/python/${PROJECT_NAME})

add_python_target(${PROJECT_NAME} ${CMAKE_CURRENT_BINARY_DIR} "" stepconf)
add_python_target(${PROJECT_NAME}_py 
		  ${CMAKE_CURRENT_BINARY_DIR}/python/${PROJECT_NAME}
		  ".py" ${PY_FILES})

install (PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/stepconf
	 DESTINATION bin)

add_glade_target(${PROJECT_NAME}_glade ${GLADE_FILES})
add_python_init_target(${PROJECT_NAME}_init)

install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/python/${PROJECT_NAME}
	DESTINATION ${PYTHON_SITE_DIR}
	COMPONENT library)

install(FILES ${GLADE_FILES} DESTINATION ${CMAKE_INSTALL_PREFIX}/share/linuxcnc)
