# CMake support for LZ4
#
# To the extent possible under law, the author(s) have dedicated all
# copyright and related and neighboring rights to this software to
# the public domain worldwide. This software is distributed without
# any warranty.
#
# For details, see <http://creativecommons.org/publicdomain/zero/1.0/>.
#
# LZ4's CMake support is maintained by Evan Nemerson; when filing
# bugs please mention @nemequ to make sure I see it.

cmake_minimum_required(VERSION 3.2)

# Parse version information
file(STRINGS "lib/lz4.h" major REGEX "^#define LZ4_VERSION_MAJOR +([0-9]+) +.*$")
string(REGEX REPLACE "^#define LZ4_VERSION_MAJOR +([0-9]+) +.*$" "\\1" major "${major}")

file(STRINGS "lib/lz4.h" minor REGEX "^#define LZ4_VERSION_MINOR +([0-9]+) +.*$")
string(REGEX REPLACE "^#define LZ4_VERSION_MINOR +([0-9]+) +.*$" "\\1" minor "${minor}")

file(STRINGS "lib/lz4.h" patch REGEX "^#define LZ4_VERSION_RELEASE +([0-9]+) +.*$")
string(REGEX REPLACE "^#define LZ4_VERSION_RELEASE +([0-9]+) +.*$" "\\1" patch "${patch}")

project(lz4 VERSION "${major}.${minor}.${patch}")

option(LZ4_BUILD_CLI_TOOLS "Indicate whether command line tools should be built." OFF)

include_directories("${CMAKE_CURRENT_LIST_DIR}/lib")

# CLI sources
set(LZ4_SOURCES
  "lib/lz4.c"
  "lib/lz4hc.c"
  "lib/lz4.h"
  "lib/lz4hc.h"
  "lib/lz4frame.c"
  "lib/lz4frame.h"
  "lib/xxhash.c")
set(LZ4_CLI_SOURCES
  "programs/bench.c"
  "programs/lz4cli.c"
  "programs/lz4io.c"
  "programs/datagen.c")

add_library(lz4 ${LZ4_SOURCES})
set(targets_to_install lz4)

if(LZ4_BUILD_CLI_TOOLS)
  add_executable(lz4cli ${LZ4_CLI_SOURCES})
  set_target_properties(lz4cli PROPERTIES OUTPUT_NAME lz4)
  target_link_libraries(lz4cli PUBLIC lz4)

  add_executable(lz4c ${LZ4_CLI_SOURCES})
  target_compile_definitions(lz4c PUBLIC ENABLE_LZ4C_LEGACY_OPTIONS)
  target_link_libraries(lz4c PUBLIC lz4)

  list(APPEND targets_to_install lz4cli lz4c)
endif()

include(GNUInstallDirs)

install(FILES
  "lib/lz4.h"
  "lib/lz4frame.h"
  "lib/lz4hc.h"
  DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")

set(generated_headers "${CMAKE_CURRENT_BINARY_DIR}/generated_headers")
set(lz4_export "${generated_headers}/${PROJECT_NAME}/LZ4_EXPORT.h")
include(GenerateExportHeader)
generate_export_header(lz4 EXPORT_FILE_NAME ${lz4_export})

target_include_directories(lz4 PUBLIC $<BUILD_INTERFACE:${generated_headers}>)

# Installation (https://github.com/forexample/package-example) {

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)

# Use:
#   * PROJECT_VERSION
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Use variables:
#   * TARGETS_EXPORT_NAME
#   * PROJECT_NAME
configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(
    TARGETS ${targets_to_install}
    EXPORT "${TARGETS_EXPORT_NAME}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(
    FILES "${lz4_export}"
    DESTINATION "${include_install_dir}/${PROJECT_NAME}"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)

# }
