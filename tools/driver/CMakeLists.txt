add_swift_host_tool(swift
  api_notes.cpp
  driver.cpp
  autolink_extract_main.cpp
  modulewrap_main.cpp
  swift_format_main.cpp
  LINK_LIBRARIES
    swiftDriver
    swiftFrontendTool
  LLVM_COMPONENT_DEPENDS
    DebugInfoCodeView
  SWIFT_COMPONENT compiler
)

# CMake doesn't support create_symlink on Windows: http://cmake.3232098.n2.nabble.com/cmake-E-create-symlink-for-Windows-td4151271.html
# TODO(hughbe/Windows): Windows does not support libedit currently. See the TODO in tools/SourceKit/tools/CMakeLists.txt
if("${CMAKE_SYSTEM_NAME}" STREQUAL "Windows")
  foreach(file swiftc;swift-format;swift-autolink-extract)
    # mklink requires that the file doesn't exist (and has no overwrite option)
    if (EXISTS "${SWIFT_RUNTIME_OUTPUT_INTDIR}/${file}.exe")
      add_custom_command(TARGET swift POST_BUILD
        COMMAND
          "del"
          "${file}.exe"
        WORKING_DIRECTORY "${SWIFT_RUNTIME_OUTPUT_INTDIR}")
    endif()

    add_custom_command(TARGET swift POST_BUILD
      COMMAND
        "mklink" "/H"
        "${file}.exe"
        "swift.exe"
      WORKING_DIRECTORY "${SWIFT_RUNTIME_OUTPUT_INTDIR}")
  endforeach()
else()
  target_link_libraries(swift edit)
  add_custom_command(TARGET swift POST_BUILD
      COMMAND "${CMAKE_COMMAND}" "-E" "create_symlink" "swift" "swiftc"
      WORKING_DIRECTORY "${SWIFT_RUNTIME_OUTPUT_INTDIR}")

  add_custom_command(TARGET swift POST_BUILD
      COMMAND "${CMAKE_COMMAND}" "-E" "create_symlink" "swift" "swift-autolink-extract"
      WORKING_DIRECTORY "${SWIFT_RUNTIME_OUTPUT_INTDIR}")

  add_custom_command(TARGET swift POST_BUILD
      COMMAND "${CMAKE_COMMAND}" "-E" "create_symlink" "swift" "swift-format"
      WORKING_DIRECTORY "${SWIFT_RUNTIME_OUTPUT_INTDIR}")
endif()

# If building as part of clang, make sure the headers are installed.
if(NOT SWIFT_BUILT_STANDALONE)
  add_dependencies(swift clang-headers)
endif()

# Platforms that have a REPL need extra libraries to be linked into the 'swift'
# binary.
include(SwiftDarwin)
if(SWIFT_HOST_VARIANT MATCHES "${SWIFT_DARWIN_VARIANTS}")
  # Link in CoreFoundation on Darwin.
  find_library(CORE_FOUNDATION NAMES CoreFoundation)
  target_link_libraries(swift ${CORE_FOUNDATION})

  # Link in arclite on Darwin.
  get_default_toolchain_dir(toolchain_dir)
  set(SWIFT_REPL_ARCLITE "${toolchain_dir}/usr/lib/arc/libarclite_${SWIFT_HOST_VARIANT}.a")
  set_property(TARGET swift APPEND_STRING PROPERTY
      LINK_FLAGS " -lobjc -Wl,-force_load,\"${SWIFT_REPL_ARCLITE}\"")
endif()

swift_install_in_component(compiler
    FILES "${SWIFT_RUNTIME_OUTPUT_INTDIR}/swiftc"
    DESTINATION "bin")
swift_install_in_component(autolink-driver
    FILES "${SWIFT_RUNTIME_OUTPUT_INTDIR}/swift-autolink-extract"
    DESTINATION "bin")
swift_install_in_component(editor-integration
    FILES "${SWIFT_RUNTIME_OUTPUT_INTDIR}/swift-format"
    DESTINATION "bin")
