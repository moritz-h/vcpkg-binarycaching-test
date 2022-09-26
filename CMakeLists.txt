cmake_minimum_required(VERSION 3.15 FATAL_ERROR)

include(FetchContent)
mark_as_advanced(FORCE
  FETCHCONTENT_BASE_DIR
  FETCHCONTENT_FULLY_DISCONNECTED
  FETCHCONTENT_QUIET
  FETCHCONTENT_UPDATES_DISCONNECTED)

find_package(Git REQUIRED)

FetchContent_Declare(vcpkg-download
  GIT_REPOSITORY https://github.com/microsoft/vcpkg.git
  GIT_TAG 2022.08.15
  GIT_SHALLOW TRUE)
FetchContent_GetProperties(vcpkg-download)
if (NOT vcpkg-download_POPULATED)
  message(STATUS "Fetch vcpkg ...")
  FetchContent_Populate(vcpkg-download)
  mark_as_advanced(FORCE
    FETCHCONTENT_SOURCE_DIR_VCPKG-DOWNLOAD
    FETCHCONTENT_UPDATES_DISCONNECTED_VCPKG-DOWNLOAD)
endif ()

set(VCPKG_BOOTSTRAP_OPTIONS "-disableMetrics")
set(VCPKG_INSTALL_OPTIONS "--clean-after-build")
set(CMAKE_TOOLCHAIN_FILE "${CMAKE_CURRENT_BINARY_DIR}/_deps/vcpkg-download-src/scripts/buildsystems/vcpkg.cmake" CACHE STRING "Vcpkg toolchain file")

# do we need to set any of these?
# X_VCPKG_NUGET_ID_PREFIX
# VCPKG_USE_NUGET_CACHE
# VCPKG_DEFAULT_BINARY_CACHE
# VCPKG_BINARY_SOURCES
# VCPKG_NUGET_REPOSITORY
# GITHUB_REPOSITORY
# GITHUB_SERVER_URL
# GITHUB_REF
# GITHUB_SHA

set(VCPKG_EXE "${CMAKE_CURRENT_BINARY_DIR}/_deps/vcpkg-download-src/vcpkg")
# force bootstrap to get the vcpkg executable
execute_process(COMMAND "${CMAKE_CURRENT_BINARY_DIR}/_deps/vcpkg-download-src/bootstrap-vcpkg.sh" "-disableMetrics")
execute_process(COMMAND "${VCPKG_EXE}"
  "fetch" "nuget"
  "--x-downloads-root=${CMAKE_CURRENT_BINARY_DIR}/_deps/nuget/"
  OUTPUT_VARIABLE NUGET_EXE_OUT)

message(STATUS "old output: ${NUGET_EXE_OUT}")
string(REGEX REPLACE ";" "\\\\;" NUGET_EXE_OUT "${NUGET_EXE_OUT}")
string(REGEX REPLACE "\n" ";" NUGET_EXE_OUT "${NUGET_EXE_OUT}")
message(STATUS "transformed output: ${NUGET_EXE_OUT}")
list(GET NUGET_EXE_OUT -2 NUGET_EXE)

message(STATUS "using vcpkg: ${VCPKG_EXE}")
message(STATUS "using nuget: ${NUGET_EXE}")

set(VCPKG_BINARY_SOURCES "nuget,https://nuget.pkg.github.com/reinago/index.json,rw")

execute_process(COMMAND "mono" "${NUGET_EXE}" "sources" "add"
  "-source" "https://nuget.pkg.github.com/reinago/index.json"
  "-storepasswordincleartext"
  "-name" '"GitHub"'
  "-username" "reinago"
  "-password" "$ENV{THE_GITHUB_TOKEN}")
execute_process(COMMAND "mono" "${NUGET_EXE}"
  "setapikey" "$ENV{THE_GITHUB_TOKEN}"
  "-source" "https://nuget.pkg.github.com/reinago/index.json")

project(vcpkg-binarycaching-test
  VERSION 0.1.0
  LANGUAGES CXX)

find_package(spdlog CONFIG REQUIRED)

add_executable(${PROJECT_NAME} main.cpp)
target_link_libraries(${PROJECT_NAME} PRIVATE spdlog::spdlog)
