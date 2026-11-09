cmake_minimum_required(VERSION 3.20)
project(lob-feed LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

option(LOB_SANITIZE "Enable AddressSanitizer and UndefinedBehaviorSanitizer" OFF)
if(LOB_SANITIZE)
  add_compile_options(-fsanitize=address,undefined -fno-omit-frame-pointer)
  add_link_options(-fsanitize=address,undefined)
endif()

add_library(lob_warnings INTERFACE)
target_compile_options(lob_warnings INTERFACE -Wall -Wextra -Wpedantic -Wconversion -Wshadow)

# Core library (headers in include/lob, sources in src)
add_library(lob_core
  src/order_book.cpp
  src/sequencer.cpp
  src/net.cpp
)
target_include_directories(lob_core PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_link_libraries(lob_core PUBLIC lob_warnings)

# Executables: enable each one once its main() exists (tasks T5 and T6)
# add_executable(replayer src/replayer_main.cpp)
# target_link_libraries(replayer PRIVATE lob_core)
# add_executable(feed_handler src/feed_handler_main.cpp)
# target_link_libraries(feed_handler PRIVATE lob_core)

# Tests
enable_testing()
include(FetchContent)
FetchContent_Declare(
  Catch2
  GIT_REPOSITORY https://github.com/catchorg/Catch2.git
  GIT_TAG v3.5.2
)
FetchContent_MakeAvailable(Catch2)
list(APPEND CMAKE_MODULE_PATH ${catch2_SOURCE_DIR}/extras)
include(Catch)

add_executable(lob_tests
  tests/test_smoke.cpp
  tests/test_protocol.cpp
  tests/test_order_book.cpp
  tests/test_sequencer.cpp
)
target_link_libraries(lob_tests PRIVATE lob_core Catch2::Catch2WithMain)
catch_discover_tests(lob_tests)
