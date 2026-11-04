cmake_minimum_required(VERSION 3.25)
project(InodeFilesystem LANGUAGES C CXX)

# The supplied course libraries are Linux x86-64 shared objects.
if(NOT CMAKE_SYSTEM_NAME STREQUAL "Linux" OR
   NOT CMAKE_SYSTEM_PROCESSOR MATCHES "^(x86_64|amd64|AMD64)$")
    message(FATAL_ERROR
        "The bundled support libraries require Linux x86-64. "
        "Build in a matching Linux environment; see README.md.")
endif()

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(GTest REQUIRED)
find_package(Threads REQUIRED)

foreach(support IN ITEMS bitmap block_store dyn_array)
    set(support_path "${CMAKE_CURRENT_SOURCE_DIR}/lib/lib${support}.so")
    if(NOT EXISTS "${support_path}")
        message(FATAL_ERROR "Missing supplied library: ${support_path}")
    endif()
    add_library(${support} SHARED IMPORTED)
    set_target_properties(${support} PROPERTIES
        IMPORTED_LOCATION "${support_path}")
endforeach()

add_library(FS SHARED src/FS.c)
target_include_directories(FS PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/include")
target_link_libraries(FS PRIVATE block_store dyn_array bitmap)

add_executable(fs_test test/tests.cpp test/tests_main.cpp)
target_link_libraries(fs_test PRIVATE FS dyn_array GTest::gtest Threads::Threads)

foreach(target IN ITEMS FS fs_test)
    target_compile_definitions(${target} PRIVATE _POSIX_C_SOURCE=200809L)
    target_compile_options(${target} PRIVATE
        -Wall -Wextra -Wshadow -Werror -Wno-sign-compare)
endforeach()

enable_testing()
file(MAKE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/test-images")
# Later tests reuse images created by earlier tests, so run the suite together.
add_test(NAME filesystem COMMAND fs_test)
set_tests_properties(filesystem PROPERTIES
    WORKING_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/test-images"
    TIMEOUT 300)
