##############################################################
#
# CMakeLists.txt - created by Cizo2000 <cizo2000@gmail.com>
#
##############################################################

cmake_minimum_required(VERSION 3.1.0)
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules ${CMAKE_MODULE_PATH})

project(sjasmplus)
set(LUA_LOCAL_LIBRARY "lua5.1")
set(TOLUA++_LOCAL_LIBRARY "tolua++")

set(SOURCES 
	sjasm/devices.cpp
	sjasm/directives.cpp
	sjasm/io_snapshots.cpp
	sjasm/io_tape.cpp
	sjasm/io_trd.cpp
	sjasm/parser.cpp
	sjasm/reader.cpp
	sjasm/sjasm.cpp
	sjasm/sjio.cpp
	sjasm/support.cpp
	sjasm/tables.cpp
	sjasm/z80.cpp
)

add_definitions(-DMAX_PATH=PATH_MAX)
add_definitions(-DCMAKE)

set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -Wall -Wabi -pedantic")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -Wall -Wabi -pedantic")

set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -s")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -s")

option(ENABLE_LUA "Enable LUA scripting support" ON)

if(ENABLE_LUA)
	message(STATUS "Looking for LUA and Tolua++")
	find_package(Lua)
	find_package(Tolua++)

	if(LUA_FOUND AND TOLUA++_FOUND)
		message(STATUS "Looking for LUA and Tolua++ - system packages enabled")
	else()
		add_subdirectory(${LUA_LOCAL_LIBRARY})
		add_subdirectory(${TOLUA++_LOCAL_LIBRARY})	
		set(LUA_LIBRARY ${LUA_LOCAL_LIBRARY})
		set(TOLUA++_LIBRARY ${TOLUA++_LOCAL_LIBRARY})
		target_include_directories(
			${LUA_LIBRARY} PUBLIC include
			${TOLUA++_LIBRARY} PUBLIC include
			)
		message(STATUS "Looking for LUA and Tolua++ - local packages enabled")
	endif(LUA_FOUND AND TOLUA++_FOUND)

	list(APPEND SOURCES 
		sjasm/lua_lpack.c
		sjasm/lua_sjasm.cpp
	)

	add_definitions(-DUSE_LUA)

	if( ${CMAKE_SYSTEM_NAME} MATCHES "Linux") 
		add_definitions(-DLUA_USE_LINUX)
	elseif(CYGWIN)
		add_definitions(-DLUA_USE_LINUX)
	elseif(WIN32)
		set(PROJECT_NAME "sjasmplus.exe")
		add_definitions(-DLUA_USE_WINDOWS)
	endif()
endif(ENABLE_LUA)

find_package(LibDL)

include_directories(sjasm)
add_executable(${PROJECT_NAME} ${SOURCES})

target_link_libraries(
	${PROJECT_NAME} 
	${LUA_LIBRARY} 
	${TOLUA++_LIBRARY}
	${LIBDL_LIBRARY}
	)

install(TARGETS ${PROJECT_NAME}
	RUNTIME DESTINATION bin)