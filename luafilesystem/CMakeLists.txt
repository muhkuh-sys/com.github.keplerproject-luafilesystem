cmake_minimum_required(VERSION 3.2.3)

PROJECT("luafilesystem")

INCLUDE(ExternalProject)

SET(VERSION_EXTERNAL "1.8.0")
SET(VERSION_PACKAGE  "2")
SET(PROJECT_VERSION  "${VERSION_EXTERNAL}.${VERSION_PACKAGE}")

# Get the VCS version for the jonchki configuration.
INCLUDE(${CMAKE_HOME_DIRECTORY}/cmake/version.cmake)

# Filter the jonchki configuration.
CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/luafilesystem.xml
               ${CMAKE_BINARY_DIR}/lua${BUILDCFG_LUA_VERSION}-luafilesystem-${PROJECT_VERSION}.xml
               @ONLY)


#----------------------------------------------------------------------------
#
# Build the project.
#

IF((${BUILDCFG_ONLY_JONCHKI_CFG} STREQUAL "OFF"))
	# Select the LUA version.
	INCLUDE(${CMAKE_HOME_DIRECTORY}/cmake/muhkuh_lua.cmake)

	SET(SUB_CMAKE_ARGS "")
	LIST(APPEND SUB_CMAKE_ARGS "-DCMAKE_C_FLAGS=${CMAKE_C_FLAGS}")
	LIST(APPEND SUB_CMAKE_ARGS "-DCMAKE_SYSTEM_NAME=${CMAKE_SYSTEM_NAME}")
	LIST(APPEND SUB_CMAKE_ARGS "-DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}")
	LIST(APPEND SUB_CMAKE_ARGS "-DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}")
	IF(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
		LIST(APPEND SUB_CMAKE_ARGS "-DCMAKE_RC_COMPILER=${CMAKE_RC_COMPILER}")
	ENDIF(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
	LIST(APPEND SUB_CMAKE_ARGS "-DLUA_LIBRARIES=${LUA_LIBRARIES}")
	LIST(APPEND SUB_CMAKE_ARGS "-DLUA_INCLUDE_DIR=${LUA_INCLUDE_DIR}")
	LIST(APPEND SUB_CMAKE_ARGS "-DCMAKE_INSTALL_PREFIX=''")
	LIST(APPEND SUB_CMAKE_ARGS "-DTEST_SCRIPT_FOLDER=${CMAKE_HOME_DIRECTORY}/cmake/tests")


	ExternalProject_Add(TARGET_luafilesystem
	                    PREFIX ${CMAKE_CURRENT_BINARY_DIR}/luafilesystem
	                    URL ${CMAKE_CURRENT_SOURCE_DIR}/luafilesystem-1_8_0.tar.gz
	                    URL_HASH SHA1=d81c8fd0a0a4c206980a9b01fa27cbd1f458031c
	                    PATCH_COMMAND "${PYTHON_EXECUTABLE}" ${CMAKE_HOME_DIRECTORY}/cmake/tools/apply_patches.py --working-folder ${CMAKE_CURRENT_BINARY_DIR}/luafilesystem/src/TARGET_luafilesystem --copy-folder ${CMAKE_CURRENT_SOURCE_DIR}/copy_over
	                    CMAKE_ARGS ${SUB_CMAKE_ARGS}
	                    TEST_COMMAND make test
	                    INSTALL_COMMAND make install DESTDIR=${CMAKE_CURRENT_BINARY_DIR}/install
	)

	#----------------------------------------------------------------------------
	#
	# Build a package for the selected platform.
	#

	SET(CPACK_OUTPUT_FILE_PREFIX ${CMAKE_BINARY_DIR}/../../repository/com/github/keplerproject/luafilesystem/${PROJECT_VERSION})
	SET(CPACK_INCLUDE_TOPLEVEL_DIRECTORY FALSE)
	SET(CPACK_PACKAGE_FILE_NAME "lua${BUILDCFG_LUA_VERSION}-luafilesystem-${PROJECT_VERSION}-${JONCHKI_PLATFORM}")

	SET(CPACK_GENERATOR "TXZ")
	INCLUDE(CPack)

	# This is the folder where the external project TARGET_luafilesystem installed the files.
	SET(SUB_INSTALL_DIR "${CMAKE_CURRENT_BINARY_DIR}/install/lib")

	IF(JONCHKI_PLATFORM_DIST_ID STREQUAL "windows")
		# Package a windows build.

		# Install the DLL without execute permissions.
		INSTALL(FILES ${SUB_INSTALL_DIR}/lfs.dll
		        DESTINATION .)

		CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/install_windows.lua
		               ${CMAKE_CURRENT_BINARY_DIR}/package/install.lua
		               @ONLY)

		INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/package/install.lua
		        DESTINATION .)
	ELSE(JONCHKI_PLATFORM_DIST_ID STREQUAL "windows")
		# Package a linux build.

		INSTALL(FILES ${SUB_INSTALL_DIR}/lfs.so
		        DESTINATION .)

		CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/install_linux.lua
		               ${CMAKE_CURRENT_BINARY_DIR}/package/install.lua
		               @ONLY)

		INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/package/install.lua
		        DESTINATION .)
	ENDIF(JONCHKI_PLATFORM_DIST_ID STREQUAL "windows")

	CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/luafilesystem.xml
	               ${CPACK_OUTPUT_FILE_PREFIX}/lua${BUILDCFG_LUA_VERSION}-luafilesystem-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.xml
	               @ONLY)


	#-----------------------------------------------------------------------------
	#
	# Create a custom target named "pack".
	# It builds the package and creates the hashes.
	#
	ADD_CUSTOM_TARGET(pack
	                  COMMAND ${CMAKE_MAKE_PROGRAM}
	                  COMMAND ${CMAKE_MAKE_PROGRAM} package
	                  COMMAND "${PYTHON_EXECUTABLE}" ${CMAKE_HOME_DIRECTORY}/cmake/tools/generate_hash.py ${CPACK_OUTPUT_FILE_PREFIX}/lua${BUILDCFG_LUA_VERSION}-luafilesystem-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.xml
	                  COMMAND "${PYTHON_EXECUTABLE}" ${CMAKE_HOME_DIRECTORY}/cmake/tools/generate_hash.py ${CPACK_OUTPUT_FILE_PREFIX}/lua${BUILDCFG_LUA_VERSION}-luafilesystem-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.tar.xz
	                  COMMENT "Pack the artifact and generate the hashes...")


	#-----------------------------------------------------------------------------
	#
	# Filter the pom.xml file.
	#
	SET(POM_ARTIFACT_ID  "lua${BUILDCFG_LUA_VERSION}-luafilesystem")
	SET(POM_GROUP_ID     "com.github.keplerproject")
	SET(POM_VERSION      "${PROJECT_VERSION}")
	SET(POM_PACKAGING    "tar.xz")
	CONFIGURE_FILE(installer/lua${BUILDCFG_LUA_VERSION}/pom.xml
	               ${CPACK_OUTPUT_FILE_PREFIX}/lua${BUILDCFG_LUA_VERSION}-luafilesystem-${PROJECT_VERSION}-${JONCHKI_PLATFORM}.pom
	               @ONLY)
ENDIF((${BUILDCFG_ONLY_JONCHKI_CFG} STREQUAL "OFF"))
