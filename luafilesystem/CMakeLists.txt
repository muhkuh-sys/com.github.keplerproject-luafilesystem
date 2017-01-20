cmake_minimum_required(VERSION 3.2.3)

PROJECT("luafilesystem")

INCLUDE(ExternalProject)

SET(VERSION_EXTERNAL "1.6.3")
SET(VERSION_PACKAGE  "1")
SET(PROJECT_VERSION  "${VERSION_EXTERNAL}.${VERSION_PACKAGE}")

# Get the VCS version for the jonchki configuration.
INCLUDE(${CMAKE_HOME_DIRECTORY}/cmake/version.cmake)

# Filter the jonchki configuration.
CONFIGURE_FILE(installer/lua5.1/luafilesystem.xml
               ${CMAKE_CURRENT_BINARY_DIR}/lua5.1-luafilesystem-${PROJECT_VERSION}.xml
               @ONLY)

#----------------------------------------------------------------------------
#
# Build the project.
#

IF((${BUILDCFG_ONLY_JONCHKI_CFG} STREQUAL "OFF"))
	# Select the LUA version.
	INCLUDE(${CMAKE_HOME_DIRECTORY}/cmake/muhkuh_lua.cmake)

	SET(LFS_CMAKE_ARGS "")
	LIST(APPEND LFS_CMAKE_ARGS "-DCMAKE_C_FLAGS=${CMAKE_C_FLAGS}")
	LIST(APPEND LFS_CMAKE_ARGS "-DCMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}")
	LIST(APPEND LFS_CMAKE_ARGS "-DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}")
	LIST(APPEND LFS_CMAKE_ARGS "-DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}")
	IF(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
		LIST(APPEND MHASH_CMAKE_ARGS "-DCMAKE_RC_COMPILER=${CMAKE_RC_COMPILER}")
	ENDIF(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
	LIST(APPEND LFS_CMAKE_ARGS "-DLUA_LIBRARIES=${LUA_LIBRARIES}")
	LIST(APPEND LFS_CMAKE_ARGS "-DLUA_INCLUDE_DIR=${LUA_INCLUDE_DIR}")
	LIST(APPEND LFS_CMAKE_ARGS "-DCMAKE_INSTALL_PREFIX=''")


	ExternalProject_Add(TARGET_luafilesystem
	                    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/luafilesystem-${VERSION_EXTERNAL}
	                    URL ${CMAKE_CURRENT_SOURCE_DIR}/luafilesystem-v_1_6_3.tar.gz
	                    URL_HASH SHA1=72bc1415e7eefb339742538505675c59bd69e6bb
	                    PATCH_COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_CURRENT_SOURCE_DIR}/templates ${CMAKE_CURRENT_BINARY_DIR}/luafilesystem-${VERSION_EXTERNAL}/src/TARGET_luafilesystem
	                    CMAKE_ARGS ${LFS_CMAKE_ARGS}
	                    TEST_COMMAND make test
	                    INSTALL_COMMAND make install DESTDIR=${CMAKE_CURRENT_BINARY_DIR}/luafilesystem-${VERSION_EXTERNAL}
	)
ENDIF((${BUILDCFG_ONLY_JONCHKI_CFG} STREQUAL "OFF"))
