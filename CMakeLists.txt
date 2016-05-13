CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

PROJECT(LuaSocket)

INCLUDE(CPack)

OPTION(WANTS_BUILD_MODULE_LIBRARY "Set to ON to build loadable module." OFF)
OPTION(WANTS_BUILD_STATIC_LIBRARY "Set to ON to build static library." ON)

set(LUA_INCLUDE_DIR "" CACHE STRING "Path to lua includes")

IF(NOT LUA_INCLUDE_DIR)
	MESSAGE(SEND_ERROR "Can't find lua headers in ${LUA_INCLUDE_DIR}")
ENDIF(NOT LUA_INCLUDE_DIR)




SET(LUAPACKAGE_CDIR "lib/lua/5.1" CACHE STRING "Path for Lua packaged platform specific things.")
SET(LUAPACKAGE_LDIR "share/lua/5.1" CACHE STRING "Path for Lua packages platform independent things.")

IF(APPLE)
	SET(LUASOCKET_LINK_FLAGS "-undefined dynamic_lookup")
	SET(LUASOCKET_C_FLAGS "-fno-common -pedantic")
	ADD_DEFINITIONS(-DLUASOCKET_DEBUG -DUNIX_HAS_SUN_LEN)
ENDIF(APPLE)
IF(WIN32)
	SET(LUASOCKET_LINK_FLAGS "wsock32")
#	SET(LUASOCKET_C_FLAGS "-fno-common -pedantic")
	ADD_DEFINITIONS(-DLUASOCKET_DEBUG "-DLUASOCKET_API=__declspec(dllexport)")
ENDIF(WIN32)


INCLUDE_DIRECTORIES(
	${LUA_INCLUDE_DIR}
)




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
