cmake_minimum_required(VERSION 3.0.0)

project(nlohmann-json
  LANGUAGES CXX
  VERSION "1.0.0")

option(JSON_BUILD_TESTS "Build tests" ON)

if(JSON_BUILD_TESTS)
  add_executable(json_unit
      src/json.hpp test/catch.hpp test/unit.cpp
  )
endif()

if ("${CMAKE_CXX_COMPILER_ID}" MATCHES "Clang")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
  set(CMAKE_CXX_FLAGS "-std=c++11")

elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  set(CMAKE_CXX_FLAGS "/EHsc")

  STRING(REPLACE "/O2" "/Od" CMAKE_CXX_FLAGS_RELEASE ${CMAKE_CXX_FLAGS_RELEASE})

  # See https://msdn.microsoft.com/fr-fr/library/aa985974.aspx?f=255&MSPPError=-2147217396
  # Avoid warnings for unsafe STL methods call
  add_definitions(-D_SCL_SECURE_NO_WARNINGS)
endif()

include_directories(
    src test
)

add_library(nlohmann-json INTERFACE)

### Install (https://github.com/forexample/package-example) ###
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY AnyNewerVersion
)

# Note: use 'targets_export_name'
configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

install(
    TARGETS nlohmann-json
    EXPORT "${targets_export_name}"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(
    DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/src/
    DESTINATION include/nlohmann
    FILES_MATCHING PATTERN "*.[ih]pp"
)

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)
