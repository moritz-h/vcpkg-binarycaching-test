cmake_minimum_required(VERSION 3.15...3.24 FATAL_ERROR)

include(FetchContent)
mark_as_advanced(FORCE
  FETCHCONTENT_BASE_DIR
  FETCHCONTENT_FULLY_DISCONNECTED
  FETCHCONTENT_QUIET
  FETCHCONTENT_UPDATES_DISCONNECTED)

find_package(Git REQUIRED)

FetchContent_Declare(vcpkg-download
  GIT_REPOSITORY https://github.com/microsoft/vcpkg.git
  GIT_TAG 2022.10.19
  GIT_SHALLOW TRUE)
FetchContent_GetProperties(vcpkg-download)
if (NOT vcpkg-download_POPULATED)
  message(STATUS "Fetch vcpkg ...")
  FetchContent_Populate(vcpkg-download)
  mark_as_advanced(FORCE
    FETCHCONTENT_SOURCE_DIR_VCPKG-DOWNLOAD
    FETCHCONTENT_UPDATES_DISCONNECTED_VCPKG-DOWNLOAD)
endif ()

set(VCPKG_OVERLAY_TRIPLETS "${CMAKE_CURRENT_LIST_DIR}/vcpkg_triplets")
set(VCPKG_BOOTSTRAP_OPTIONS "-disableMetrics")
set(VCPKG_INSTALL_OPTIONS "--clean-after-build" "--no-print-usage")
set(CMAKE_TOOLCHAIN_FILE "${CMAKE_CURRENT_BINARY_DIR}/_deps/vcpkg-download-src/scripts/buildsystems/vcpkg.cmake" CACHE STRING "Vcpkg toolchain file")

set(ENV{VCPKG_FORCE_DOWNLOADED_BINARIES} ON)

project(vcpkg-binarycaching-test
  VERSION 0.1.0
  LANGUAGES CXX)

# Allow CI dependency build to stop here.
if (STOP_AFTER_VCPKG)
  return()
endif ()

find_package(spdlog CONFIG REQUIRED)

add_executable(${PROJECT_NAME} main.cpp)
target_link_libraries(${PROJECT_NAME} PRIVATE spdlog::spdlog)
