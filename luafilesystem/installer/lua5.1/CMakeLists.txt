CMAKE_MINIMUM_REQUIRED(VERSION 3.0)

PROJECT("com.github.keplerproject-luafilesystem")

INCLUDE(../../../cmake/version.cmake)

SET(INSTALL_WIN32 ${CMAKE_CURRENT_SOURCE_DIR}/../../../build/windows32/lua5.1/luafilesystem/install)
SET(INSTALL_WIN64 ${CMAKE_CURRENT_SOURCE_DIR}/../../../build/windows64/lua5.1/luafilesystem/install)

# Get the project version. Both the WIN32 and the WIN64 folder have the same version file.
INCLUDE(${INSTALL_WIN32}/com.github.keplerproject-luafilesystem.version)
SET(PROJECT_VERSION ${com.github.keplerproject-luafilesystem_VERSION})

SET(CPACK_OUTPUT_FILE_PREFIX targets/jonchki/repository/com/github/keplerproject/luafilesystem/${PROJECT_VERSION})
SET(CPACK_INCLUDE_TOPLEVEL_DIRECTORY FALSE)
SET(CPACK_PACKAGE_FILE_NAME "lua5.1-luafilesystem-${PROJECT_VERSION}")

SET(CPACK_GENERATOR "ZIP")
INCLUDE(CPack)


#-----------------------------------------------------------------------------
#
# Install the 32bit files.
#
INSTALL(FILES ${INSTALL_WIN32}/lua_plugins/lfs.dll
        DESTINATION windows_x86/lua_plugins
)


#-----------------------------------------------------------------------------
#
# Install the 64bit files.
#
INSTALL(FILES ${INSTALL_WIN64}/lua_plugins/lfs.dll
        DESTINATION windows_x64/lua_plugins
)


#-----------------------------------------------------------------------------

INSTALL(FILES install.lua
        DESTINATION .)


CONFIGURE_FILE(bitop.xml
               ${CPACK_OUTPUT_FILE_PREFIX}/lua5.1-filesystem-${PROJECT_VERSION}.xml
               @ONLY)
