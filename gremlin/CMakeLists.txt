cmake_minimum_required(VERSION 2.8 )
project(gremlin)

execute_process(COMMAND ${CMAKE_COMMAND} 
	-E make_directory ${CMAKE_CURRENT_BINARY_DIR}/python/${PROJECT_NAME})

add_python_target(${PROJECT_NAME} ${CMAKE_CURRENT_BINARY_DIR} "" gremlin-run)
add_python_target(${PROJECT_NAME}_py ${CMAKE_CURRENT_BINARY_DIR}/python/${PROJECT_NAME}
		  ".py" gremlin)

install (PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/gremlin-run
	 DESTINATION bin)

add_python_init_target(${PROJECT_NAME}_init)

install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/python
	DESTINATION ${PYTHON_SITE_DIR}
	COMPONENT library)
