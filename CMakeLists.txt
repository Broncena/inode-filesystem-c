cmake_minimum_required(VERSION 3.25)
project(FS)
set(CMAKE_COLOR_MAKEFILE ON)
if(APPLE)
    # Make policy CMP0042 warning go away
    # even though I still don't quite understand it
    set(CMAKE_MACOSX_RPATH ON)
endif(APPLE)

# Specific paths for Hellbender
set(CS4520_INCLUDE_DIRS "/cluster/pixstor/class/cs4520/include")
set(CS4520_LIB_DIRS "/cluster/pixstor/class/cs4520/lib")
include_directories(${CS4520_INCLUDE_DIRS})
link_directories(${CS4520_LIB_DIRS})

include_directories(include)
include_directories(${GTEST_INCLUDE_DIRS} include)

link_directories(${CMAKE_CURRENT_SOURCE_DIR}/lib)

#add_library(bitmap SHARED src/bitmap.c)
#add_library(block_store SHARED src/block_store.c)
#add_library(dyn_array SHARED src/dyn_array.c)
add_library(FSTest SHARED test/tests.cpp)
#find_package(GTest REQUIRED)
set(SHARED_FLAGS " -Wall -Wextra -Wshadow -Werror -g -D_POSIX_C_SOURCE=200809L -Wno-sign-compare")
set(CMAKE_CXX_FLAGS " ${SHARED_FLAGS}")
set(CMAKE_C_FLAGS " ${SHARED_FLAGS}")

add_library(FS SHARED src/FS.c)
set_target_properties(FS PROPERTIES POSITION_INDEPENDENT_CODE ON)
target_link_libraries(FS block_store dyn_array bitmap)

add_executable(fs_test test/tests_main.cpp)
target_compile_definitions(fs_test PRIVATE)
target_link_libraries(fs_test FSTest FS gtest pthread)
