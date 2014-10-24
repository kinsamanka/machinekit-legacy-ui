cmake_minimum_required(VERSION 2.8 )
project(gscreen)

set(PY_FILES mdi
	emc_interface
	preferences
	keybindings)

set(GLADE_FILES gscreen.glade gscreen2.glade)

execute_process(COMMAND ${CMAKE_COMMAND} 
	-E make_directory ${CMAKE_CURRENT_BINARY_DIR}/python/${PROJECT_NAME})

add_python_target(${PROJECT_NAME} ${CMAKE_CURRENT_BINARY_DIR} "" gscreen)
add_python_target(${PROJECT_NAME}_py 
		  ${CMAKE_CURRENT_BINARY_DIR}/python/${PROJECT_NAME}
		  ".py" ${PY_FILES})

install (PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/gscreen
	 DESTINATION bin)

add_glade_target(${PROJECT_NAME}_glade ${GLADE_FILES})
add_python_init_target(${PROJECT_NAME}_init)

install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/python/${PROJECT_NAME}
	DESTINATION ${PYTHON_SITE_DIR}
	COMPONENT library)

install(FILES ${GLADE_FILES} DESTINATION ${CMAKE_INSTALL_PREFIX}/share/linuxcnc)
