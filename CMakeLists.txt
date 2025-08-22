cmake_minimum_required(VERSION 3.20)
project(MyProject VERSION 1.0 LANGUAGES CXX)

# Export compile_commands.json for IDE/LSP tooling
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Default build type if not specified
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE "Debug" CACHE STRING "Choose the type of build." FORCE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "RelWithDebInfo" "MinSizeRel")
endif()

# C++ standard (no compiler-specific extensions)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Options for common workflows
option(ENABLE_WARNINGS "Enable compiler warnings" ON)
option(ENABLE_WARNINGS_AS_ERRORS "Treat warnings as errors" OFF)
option(ENABLE_SANITIZERS "Enable Address/UBSan (Debug-only, non-MSVC)" OFF)
option(ENABLE_LTO "Enable Link-Time Optimization in Release" OFF)
option(ENABLE_CLANG_TIDY "Run clang-tidy during build if available" OFF)

# Collect sources (simple template approach)
file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")

add_executable(${PROJECT_NAME} ${SOURCES})

# Public include directory for headers in include/
# Private include src/ for internal headers if used
target_include_directories(${PROJECT_NAME}
  PUBLIC  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/inc>"
  PRIVATE "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>"
)

# Require C++20 features at the target level as well
target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_20)

# Cross-platform warnings configuration
function(enable_target_warnings target)
  if(MSVC)
    target_compile_options(${target} PRIVATE /W4)
    target_compile_definitions(${target} PRIVATE _CRT_SECURE_NO_WARNINGS NOMINMAX)
    if(ENABLE_WARNINGS_AS_ERRORS)
      target_compile_options(${target} PRIVATE /WX)
    endif()
  else()
    target_compile_options(${target} PRIVATE -Wall -Wextra -Wpedantic)
    if(ENABLE_WARNINGS_AS_ERRORS)
      target_compile_options(${target} PRIVATE -Werror)
    endif()
  endif()
endfunction()

if(ENABLE_WARNINGS)
  enable_target_warnings(${PROJECT_NAME})
endif()

# Sanitizers: Address + Undefined in Debug for Clang/GCC
if(ENABLE_SANITIZERS AND NOT MSVC)
  set(SAN_FLAGS "-fsanitize=address,undefined")
  if(CMAKE_BUILD_TYPE MATCHES "^Debug$")
    target_compile_options(${PROJECT_NAME} PRIVATE ${SAN_FLAGS})
    target_link_options(${PROJECT_NAME} PRIVATE ${SAN_FLAGS})
  endif()
endif()

# LTO / IPO for Release builds
if(ENABLE_LTO)
  include(CheckIPOSupported)
  check_ipo_supported(RESULT ipo_supported OUTPUT ipo_error)
  if(ipo_supported)
    set_property(TARGET ${PROJECT_NAME} PROPERTY INTERPROCEDURAL_OPTIMIZATION_RELEASE TRUE)
  else()
    message(WARNING "IPO/LTO not supported: ${ipo_error}")
  endif()
endif()

# clang-tidy integration (reads rules from .clang-tidy in repo root)
if(ENABLE_CLANG_TIDY)
  find_program(CLANG_TIDY_EXE NAMES clang-tidy)
  if(CLANG_TIDY_EXE)
    set(CMAKE_CXX_CLANG_TIDY "${CLANG_TIDY_EXE}")
  else()
    message(WARNING "clang-tidy requested but not found")
  endif()
endif()

# Installation (binary into standard bindir)
include(GNUInstallDirs)
install(TARGETS ${PROJECT_NAME}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

# Testing
include(CTest)
if(BUILD_TESTING)
  enable_testing()
  if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/tests/CMakeLists.txt")
    add_subdirectory(tests)
  endif()
endif()
