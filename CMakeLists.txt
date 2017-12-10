cmake_minimum_required(VERSION 2.8.12)
project(zeromq-shared-static)
list(APPEND CMAKE_PREFIX_PATH ${CMAKE_BINARY_DIR}/deps/${CMAKE_SYSTEM_NAME}/share/cmake/ZeroMQ)
find_package(ZeroMQ QUIET)
set (ZMQ_FORCE_BUILD TRUE CACHE BOOL "Force Build ZMQ")

set (ZMQ_GIT_TAG_BROKEN 0298d037d0665cbb6b4b6f8fc8bc38ae296669b3 CACHE STRING "Broken git tag")
set (ZMQ_GIT_TAG_WORKING 5264d49eab79c427c8b8c274af69a4ffd42d364d CACHE  STRING "Working git tag")

set (ZMQ_GIT_TAG ${ZMQ_GIT_TAG_BROKEN} CACHE STRING "Git tag to checkout" )
		
if(ZMQ_FORCE_BUILD OR NOT ZeroMQ_FOUND )
	#setup dependencies
	configure_file(CMakeLists-zeromq.txt ${CMAKE_BINARY_DIR}/deps-build/CMakeLists.txt @ONLY)
	execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
	  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/deps-build )
	execute_process(COMMAND ${CMAKE_COMMAND} --build . --clean-first --config Debug
	  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/deps-build )
	execute_process(COMMAND ${CMAKE_COMMAND} --build . --clean-first --config Release
	  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}/deps-build )
	set (ZMQ_FORCE_BUILD FALSE CACHE BOOL "Force Build ZMQ" FORCE)
endif()
find_package(ZeroMQ REQUIRED)
include_directories(${ZeroMQ_INCLUDE_DIR})
add_executable(${PROJECT_NAME} WIN32 HelloWorld.cc)
target_link_libraries(${PROJECT_NAME} libzmq-static)
target_link_libraries(${PROJECT_NAME} "iphlpapi.lib" "psapi.lib" "ws2_32.lib")
add_definitions(-DZMQ_STATIC)
target_include_directories(${PROJECT_NAME} PUBLIC ${ZeroMQ_INCLUDE_DIR})
set_target_properties(${PROJECT_NAME}  PROPERTIES LINK_FLAGS "/SUBSYSTEM:CONSOLE")
set_target_properties(${PROJECT_NAME}  PROPERTIES COMPILE_DEFINITIONS "_CONSOLE")
