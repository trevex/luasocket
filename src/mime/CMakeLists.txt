# Ugh. I'm very irritated that the Luasocket people rename the modules to "core". (No prefix, .dll/.so suffix). This makes life difficult because I can't build the libraries directly with the name because there are two libraries (socket, mime) so the names conflict with each other.
# Furthermore, CMake doesn't offer good support for renaming TARGETS on install.
# The easiest way to work around this problem was create a new subdirectory just for the mime plugin and build directly to the name 'core'.


SET(luasocket_MIME
		${LuaSocket_SOURCE_DIR}/src/mime.h
		${LuaSocket_SOURCE_DIR}/src/mime.c
)
SET(luamime_RESOURCE_FILES
#		${LuaSocket_SOURCE_DIR}/src/mime.lua
)


IF(WANTS_BUILD_SHARED_LIBRARY)
	ADD_LIBRARY(luamime_library_dynamic MODULE ${luasocket_PUBLIC_HEADERS} ${luasocket_MIME})
	SET_TARGET_PROPERTIES(luamime_library_dynamic PROPERTIES
		PREFIX ""
		OUTPUT_NAME "core"
	)
	SET_TARGET_PROPERTIES(luamime_library_dynamic PROPERTIES COMPILE_FLAGS "${LUASOCKET_C_FLAGS}")
	TARGET_LINK_LIBRARIES(luamime_library_dynamic ${LUA_LIBRARIES} ${LUASOCKET_LINK_FLAGS})
ENDIF(WANTS_BUILD_SHARED_LIBRARY)
#IF(WANTS_BUILD_STATIC_LIBRARY)
	ADD_LIBRARY(luamime_library_static STATIC ${luasocket_PUBLIC_HEADERS} ${luasocket_MIME})
	SET_TARGET_PROPERTIES(luamime_library_static PROPERTIES OUTPUT_NAME "mime")
	SET_TARGET_PROPERTIES(luamime_library_static PROPERTIES COMPILE_FLAGS "${LUASOCKET_C_FLAGS}")
	TARGET_LINK_LIBRARIES(luamime_library_static ${LUA_LIBRARIES} ${LUASOCKET_LINK_FLAGS})
#ENDIF(WANTS_BUILD_STATIC_LIBRARY)

INSTALL(TARGETS
	luamime_library_static
	RUNTIME DESTINATION bin
	LIBRARY DESTINATION lib
	ARCHIVE DESTINATION lib
	#	ARCHIVE DESTINATION lib/static
)

IF(WANTS_BUILD_SHARED_LIBRARY)
	INSTALL(TARGETS luamime_library_dynamic DESTINATION ${LUAPACKAGE_CDIR}/mime)
ENDIF(WANTS_BUILD_SHARED_LIBRARY)

INSTALL(FILES ${luamime_RESOURCE_FILES} DESTINATION ${LUAPACKAGE_LDIR}/mime)


