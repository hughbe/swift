set(SOURCEKITD_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/sourcekitd)

include_directories(
  ${SOURCEKITD_SOURCE_DIR}/include
)

add_swift_lib_subdirectory(sourcekitd)
add_swift_tool_subdirectory(sourcekitd-test)

# Requires libedit (which is not available on Windows)
# TODO (hughbe/Windows): http://thrysoee.dk/editline/ as an external solution?
# Also, see FIXME in Repl.cpp:
# FIXME: Support REPL on non-Apple platforms. Ubuntu 14.10's editline does not
# include the wide character entry points needed by the REPL yet.
if(NOT "${CMAKE_SYSTEM_NAME}" STREQUAL "Windows")
  add_swift_tool_subdirectory(sourcekitd-repl)
endif()
add_swift_tool_subdirectory(complete-test)
