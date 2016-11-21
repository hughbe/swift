add_swift_host_tool(swift-remoteast-test
  swift-remoteast-test.cpp
  LINK_LIBRARIES
    swiftFrontendTool
    swiftRemoteAST
  SWIFT_COMPONENT tools
)

set_target_properties(swift-remoteast-test PROPERTIES ENABLE_EXPORTS 1)

# TODO(hughbe/Windows): Windows does not support libedit currently. See the TODO in tools/SourceKit/tools/CMakeLists.txt
if(NOT "${CMAKE_SYSTEM_NAME}" STREQUAL "Windows")
  target_link_libraries(swift-remoteast-test edit)
endif()

# If building as part of clang, make sure the headers are installed.
if(NOT SWIFT_BUILT_STANDALONE)
  add_dependencies(swift clang-headers)
endif()

