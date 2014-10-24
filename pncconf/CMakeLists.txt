cmake_minimum_required(VERSION 2.8 )
project(pncconf)

set(GLADE_FILES pncconf.glade)

add_python_target(${PROJECT_NAME} ${CMAKE_CURRENT_BINARY_DIR} "" pncconf)

install (PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/pncconf
	 DESTINATION bin)

add_glade_target(${PROJECT_NAME}_glade ${GLADE_FILES})

install(FILES ${GLADE_FILES} DESTINATION ${CMAKE_INSTALL_PREFIX}/share/linuxcnc)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/pncconf-help
	DESTINATION ${CMAKE_INSTALL_PREFIX}/share/doc/linuxcnc)
