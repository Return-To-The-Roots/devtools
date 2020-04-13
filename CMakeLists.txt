if(MSVC)
  # Append msvcContribDir to CMAKE_PREFIX_PATH
  # Set CMAKE_LIBRARY_ARCHITECTURE and CONTRIB_DLL_DIR
	if(CMAKE_SIZEOF_VOID_P EQUAL 4)
		set(CMAKE_LIBRARY_ARCHITECTURE "x86" CACHE INTERNAL "")
	else()
		set(CMAKE_LIBRARY_ARCHITECTURE "x64" CACHE INTERNAL "")
	endif()
	MESSAGE(STATUS "Building for MSVC: ${CMAKE_LIBRARY_ARCHITECTURE}")

	set(msvcContribDir ${CMAKE_CURRENT_SOURCE_DIR}/msvc)
	set(_dllDir "${msvcContribDir}/bin/${CMAKE_LIBRARY_ARCHITECTURE}")
    # Add dlls to the gathered ones
	if(NOT EXISTS ${_dllDir}/libcurl.dll)
		message(WARNING "Folder with DLLs not found in ${_dllDir}. You may not be able to execute directly from VS")
	else()
		file(GLOB CONTRIB_DLLS ${_dllDir}/*.dll)
		include(GatherDll)
		gather_dll_add(${CONTRIB_DLLS})
	endif()
	list(APPEND CMAKE_PREFIX_PATH ${msvcContribDir})
	list(APPEND CMAKE_PROGRAM_PATH ${msvcContribDir}/buildTools ${_dllDir})
	set(CMAKE_PREFIX_PATH ${CMAKE_PREFIX_PATH} PARENT_SCOPE)
	set(CMAKE_PROGRAM_PATH ${CMAKE_PROGRAM_PATH} PARENT_SCOPE)
ENDIF()

################################################################################
# LUA
################################################################################

# Here we set hints to lua libraries used for official builds.
# Normally those should be on the build server but we make them available here for convenience.
# As this will be appended to CMAKE_PREFIX_PATH one can use other versions by setting
# that appropriately

set(_contrib_lua_path ${CMAKE_CURRENT_LIST_DIR}/lua)
set(_contrib_lua_libpath "")

if(MSVC)
	# Library is in contrib archive which is in prefix path.
	# We just need the includes (which we did not duplicate in the msvc-contrib)
	list(APPEND CMAKE_PREFIX_PATH ${_contrib_lua_path})
	set(CMAKE_PREFIX_PATH ${CMAKE_PREFIX_PATH} PARENT_SCOPE)
elseif(WIN32 OR CYGWIN)
	if(CMAKE_SIZEOF_VOID_P EQUAL 8)
		set(_contrib_lua_libpath ${_contrib_lua_path}/win64)
  else()
		set(_contrib_lua_libpath ${_contrib_lua_path}/win32)
	endif()
elseif(CMAKE_SYSTEM_NAME STREQUAL "Linux")
	if(CMAKE_SIZEOF_VOID_P EQUAL 8)
		set(_contrib_lua_libpath ${_contrib_lua_path}/linux)
	endif()
elseif(APPLE)
	set(_contrib_lua_libpath ${_contrib_lua_path}/mac)
endif()

# Set only if we have a library for this arch
if(_contrib_lua_libpath)
	if(NOT EXISTS ${_contrib_lua_path}/include/lua.h)
		message(WARNING "Could not find lua.h in contrib")
	endif()
	# For cross compilation we must set CMAKE_FIND_ROOT_PATH instead
	if(CMAKE_CROSSCOMPILING)
		list(APPEND CMAKE_FIND_ROOT_PATH ${_contrib_lua_path} ${_contrib_lua_libpath})
		set(CMAKE_FIND_ROOT_PATH ${CMAKE_FIND_ROOT_PATH} PARENT_SCOPE)
	else()
		list(APPEND CMAKE_PREFIX_PATH ${_contrib_lua_path} ${_contrib_lua_libpath})
		set(CMAKE_PREFIX_PATH ${CMAKE_PREFIX_PATH} PARENT_SCOPE)
	endif()
endif()