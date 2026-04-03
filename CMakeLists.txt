cmake_minimum_required(VERSION 3.10)
project(pitch_detection VERSION 1.0 LANGUAGES CXX)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# use this for dependency graph generation
#set_property(GLOBAL PROPERTY GRAPHVIZ_EXPORT_TARGETS TRUE)

if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang" OR CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    set(CMAKE_CXX_FLAGS "-Wall -Wextra -ansi -pedantic -fext-numeric-literals -fopenmp")
    set(CMAKE_CXX_FLAGS_DEBUG "-g")
    set(CMAKE_CXX_FLAGS_RELEASE "-O3 -march=native -flto")
endif()

include_directories(include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/test)

find_package(unofficial-linkotec-ffts CONFIG REQUIRED)
# Assuming FFTS does not have a built-in find module

# Add the library target
file(GLOB_RECURSE LIB_SOURCES "src/*.cpp")
add_library(pitch_detection SHARED ${LIB_SOURCES})
find_path(MLPACK_INCLUDE_DIRS "mlpack.hpp")
target_include_directories(pitch_detection PRIVATE ${MLPACK_INCLUDE_DIRS})
target_include_directories(pitch_detection PRIVATE ${FFTS_INCLUDE_DIR})
target_link_libraries(pitch_detection PRIVATE ${FFTS_LIBRARY_RELEASE})

find_package(gflags QUIET)
find_package(libnyquist QUIET)
find_package(PkgConfig QUIET)

if(gflags_FOUND AND libnyquist_FOUND)
    file(GLOB WAV_ANALYZER_SOURCES "wav_analyzer/*.cpp")
    add_executable(wav_analyzer ${WAV_ANALYZER_SOURCES})
    target_link_libraries(wav_analyzer PRIVATE
        pitch_detection
        ${GFLAGS_LIBRARIES}
        libnyquist)

    target_include_directories(wav_analyzer PRIVATE
        ${GFLAGS_INCLUDE_DIRS}
        /usr/src/libnyquist-repo/include/)

    link_directories(
        ${GFLAGS_LIBRARY_DIRS}
        ${LIBYNQUIST_LIBRARY_DIRS})
endif()

include(CTest)
find_package(GTest QUIET)
find_package(lib QUIET)
if(GTEST_FOUND)
    enable_testing()
    file(GLOB TEST_SOURCES "test/test*.cpp" "test/util.cpp")
    add_executable(pitch_tests ${TEST_SOURCES})
    target_link_libraries(pitch_tests PRIVATE
        pitch_detection
        GTest::GTest
        GTest::Main)
endif()

find_package(benchmark QUIET)
if(benchmark_FOUND)
    file(GLOB BENCH_SOURCES "test/bench.cpp" "test/util.cpp")
    add_executable(pitch_bench ${BENCH_SOURCES})
    target_link_libraries(pitch_bench PRIVATE
        pitch_detection
        benchmark::benchmark)
endif()

install(TARGETS pitch_detection LIBRARY DESTINATION lib)
install(FILES include/pitch_detection.h DESTINATION include)
