add_swift_host_tool(swift-ide-test
  swift-ide-test.cpp
  ModuleAPIDiff.cpp
  XMLValidator.cpp
  LINK_LIBRARIES
    swiftDriver
    swiftFrontend
    swiftIDE
  LLVM_COMPONENT_DEPENDS
    DebugInfoCodeView
  SWIFT_COMPONENT tools
)

# If libxml2 is available, make it available for swift-ide-test.
# libxml2 requires a lot of effort to build on Windows (requires GNUWin32 packages such as iconv)
# TODO: we should try to get this working on Windows, but let's disable it until a good way to include libxml2 is possible
if(NOT "${CMAKE_SYSTEM_NAME}" STREQUAL "Windows" AND SWIFT_HAVE_LIBXML)
  include_directories(SYSTEM ${LIBXML2_INCLUDE_DIR})
  target_link_libraries(swift-ide-test ${LIBXML2_LIBRARIES})
  add_definitions(-DSWIFT_HAVE_LIBXML="1")
endif()

# Create a symlink for swift-api-dump.py in the bin directory

# CMake doesn't support create_symlink on Windows: http://cmake.3232098.n2.nabble.com/cmake-E-create-symlink-for-Windows-td4151271.html
if ("${CMAKE_SYSTEM_NAME}" STREQUAL "Windows")
  if (EXISTS "${SWIFT_RUNTIME_OUTPUT_INTDIR}/swift-api-dump.py")
    add_custom_command(TARGET swift-ide-test POST_BUILD
      COMMAND
        "del"
        "swift-api-dump.py"
      WORKING_DIRECTORY "${SWIFT_RUNTIME_OUTPUT_INTDIR}")
  endif()
  add_custom_command(TARGET swift-ide-test POST_BUILD
      COMMAND
        "mklink" "/H"
        "\"${SWIFT_RUNTIME_OUTPUT_INTDIR}/swift-api-dump.py\""
        "\"${SWIFT_SOURCE_DIR}/utils/swift-api-dump.py\""
      COMMENT "Creating development symlink for swift-api-dump.py")
else()
  add_custom_command(TARGET swift-ide-test POST_BUILD
      COMMAND
        "${CMAKE_COMMAND}" "-E" "create_symlink"
        "${SWIFT_SOURCE_DIR}/utils/swift-api-dump.py"
        "${SWIFT_RUNTIME_OUTPUT_INTDIR}/swift-api-dump.py"
      COMMENT "Creating development symlink for swift-api-dump.py")
endif()