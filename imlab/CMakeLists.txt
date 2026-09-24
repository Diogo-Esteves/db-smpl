cmake_minimum_required(VERSION 3.20)
project(imlab CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Debug)
endif()

add_compile_options(-Wall -Wextra)

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  add_compile_options(-fsanitize=address,undefined -fno-omit-frame-pointer -g)
  add_link_options(-fsanitize=address,undefined)
endif()

# --- library: storage layer, TPC-C hand-written transactions, etc. ---
# INTERFACE until Week 1 adds real .cc sources (storage structs, loader, indexes).
add_library(imlab_core INTERFACE)
target_include_directories(imlab_core INTERFACE include)

# --- tests ---
find_package(GTest REQUIRED)
enable_testing()

add_executable(imlab_tests
  test/dummy_test.cc
)
target_link_libraries(imlab_tests PRIVATE imlab_core GTest::gtest GTest::gtest_main)
target_include_directories(imlab_tests PRIVATE include)

include(GoogleTest)
gtest_discover_tests(imlab_tests)
