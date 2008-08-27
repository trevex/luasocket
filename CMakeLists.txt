IF(APPLE)
	CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
ELSE(APPLE)
	CMAKE_MINIMUM_REQUIRED(VERSION 2.4)
ENDIF(APPLE)

PROJECT(LuaSocket)

IF(USING_LUA_MULTI_SET)
	IF(NOT LUA_LIBRARIES)
		IF(lua_library_dynamic)
			SET(LUA_LIBRARIES lua_library_dynamic)
		ELSE(lua_library_dynamic)
			SET(LUA_LIBRARIES lua_library_static)
		ENDIF(lua_library_dynamic)
	ENDIF(NOT LUA_LIBRARIES)
	IF(NOT LUA_INCLUDE_DIR)
		SET(LUA_INCLUDE_DIR ${Lua_SOURCE_DIR}/src)
	ENDIF(NOT LUA_INCLUDE_DIR)
ELSE(USING_LUA_MULTI_SET)
	FIND_PACKAGE(Lua51 REQUIRED)
ENDIF(USING_LUA_MULTI_SET)

OPTION(WANTS_BUILD_SHARED_LIBRARY "Set to ON to build dynamic library." ON)
#OPTION(WANTS_BUILD_STATIC_LIBRARY "Set to ON to build static library." ON)
OPTION(WANTS_BUILD_FRAMEWORK "Set to ON to build framework instead of dylib. Only valid if BUILD_SHARED_LIBRARY is ON an is OS X." ON)

SET(LUAPACKAGE_CDIR "lib/lua/5.1" CACHE STRING "Path for Lua packaged platform specific things.")
SET(LUAPACKAGE_LDIR "share/lua/5.1" CACHE STRING "Path for Lua packages platform independent things.")

IF(APPLE)
	SET(LUASOCKET_LINK_FLAGS "-undefined dynamic_lookup")
	SET(LUASOCKET_C_FLAGS "-fno-common -pedantic")
	ADD_DEFINITIONS(-DLUASOCKET_DEBUG -DUNIX_HAS_SUN_LEN)
ENDIF(APPLE)


INCLUDE_DIRECTORIES(
	${LUA_INCLUDE_DIR}
)



#IF(WANTS_BUILD_SHARED_LIBRARY AND WANTS_BUILD_FRAMEWORK)
#	SET_TARGET_PROPERTIES(luasocket_library_dynamic PROPERTIES
#		FRAMEWORK TRUE
#		# FRAMEWORK_VERSION 5.1
#		#  PRIVATE_HEADER "fooPrivate.h;fooBoth.h"
#		PUBLIC_HEADER "${luasocket_PUBLIC_HEADERS}"
		# RESOURCE "${RESOURCE_FILES}"
#		INSTALL_NAME_DIR "@executable_path/../Frameworks"
#		BUILD_WITH_INSTALL_RPATH 1 # FIXME: User option or Xcode=1 
#	)
#ENDIF(WANTS_BUILD_SHARED_LIBRARY AND WANTS_BUILD_FRAMEWORK)

#SET_TARGET_PROPERTIES(luasocket_library_dynamic PROPERTIES
#	PUBLIC_HEADER "${luasocket_PUBLIC_HEADERS}"
#	RESOURCE "${luasocket_RESOURCE_FILES}"
#)	


IF(NOT USING_LUA_MULTI_SET)
# For uninstall (needs cmake_uninstall.cmake.in in the top-level directory)
CONFIGURE_FILE(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
  IMMEDIATE @ONLY)
ADD_CUSTOM_TARGET(uninstall
  "${CMAKE_COMMAND}" -P "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake")
ENDIF(NOT USING_LUA_MULTI_SET)



# Set defaults for Universal Binaries. We want 32-bit Intel/PPC on 10.4 
# and 32/64-bit Intel/PPC on >= 10.5. Anything <= 10.3 doesn't support.
IF(APPLE)
    # These are just defaults/recommendations, but how we want to build
    # out of the box. But the user needs to be able to change these options.
    # So we must only set the values the first time CMake is run, or we 
    # will overwrite any changes the user sets.
    # FORCE is used because the options are not reflected in the UI otherwise.
    # Seems like a good place to add version specific compiler flags too.
    IF(NOT LUA_CONFIG_HAS_BEEN_RUN_BEFORE)
        # This is really fragile, but CMake doesn't provide the OS system 
        # version information we need. (Darwin versions can be changed 
        # independently of OS X versions.)
        # It does look like CMake handles the CMAKE_OSX_SYSROOT automatically.
        IF(EXISTS /Developer/SDKs/MacOSX10.5.sdk)
            SET(CMAKE_OSX_ARCHITECTURES "ppc;i386;ppc64;x86_64" CACHE STRING "Build architectures for OSX" FORCE)
            SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mmacosx-version-min=10.5" CACHE STRING "Flags used by the compiler during all build types." FORCE)
        ELSE(EXISTS /Developer/SDKs/MacOSX10.5.sdk)
            IF(EXISTS /Developer/SDKs/MacOSX10.4u.sdk)
                SET(CMAKE_OSX_ARCHITECTURES "ppc;i386" CACHE STRING "Build architectures for OSX" FORCE)
                SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mmacosx-version-min=10.4" CACHE STRING "Flags used by the compiler during all build types." FORCE)
            ELSE(EXISTS /Developer/SDKs/MacOSX10.4u.sdk)
                # No Universal Binary support
                # Should break down further to set the -mmacosx-version-min,
                # but the SDK detection is too unreliable here.
            ENDIF(EXISTS /Developer/SDKs/MacOSX10.4u.sdk)
        ENDIF(EXISTS /Developer/SDKs/MacOSX10.5.sdk)
    ENDIF(NOT LUA_CONFIG_HAS_BEEN_RUN_BEFORE)
ENDIF(APPLE)

# This needs to be run very last so other parts of the scripts can take
# advantage of this.
IF(NOT LUA_CONFIG_HAS_BEEN_RUN_BEFORE)
    SET(LUA_CONFIG_HAS_BEEN_RUN_BEFORE 1 CACHE INTERNAL "Flag to track whether this is the first time running CMake or if CMake has been configured before")
ENDIF(NOT LUA_CONFIG_HAS_BEEN_RUN_BEFORE)


ADD_SUBDIRECTORY(src)