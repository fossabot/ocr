cmake_minimum_required(VERSION 3.18)
project(ocr VERSION 0.1.0 LANGUAGES CXX)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

string(TIMESTAMP NOW "%Y-%m-%dT%H:%M:%SZ")
set(CURRENT_VERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}-${NOW}")
message("version: ${CURRENT_VERSION}")

if(MSVC)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
endif()

if(NOT CMAKE_BUILD_TYPE OR CMAKE_BUILD_TYPE MATCHES "Release")
    set(CMAKE_BUILD_TYPE Release)
elseif(CMAKE_BUILD_TYPE MATCHES "Debug")
    set(CMAKE_BUILD_TYPE Debug)
else()
    message(FATAL_ERROR "Available build types: Debug and Release.")
endif(NOT CMAKE_BUILD_TYPE OR CMAKE_BUILD_TYPE MATCHES "Release")

option(ENABLE_TESTS "Enable tests" ON)
if (ENABLE_TESTS)
    enable_testing()
    add_subdirectory(test EXCLUDE_FROM_ALL)
endif()

option(ENABLE_BENCHMARK "Enable benchmark" ON)
if (ENABLE_BENCHMARK)
    add_subdirectory(benchmark EXCLUDE_FROM_ALL)
endif()

option(ENABLE_EXAMPLE "Enable example" ON)
if (ENABLE_EXAMPLE)
    add_subdirectory(example EXCLUDE_FROM_ALL)
endif()

option(ENABLE_DOC "Enable documentation" ON)
if (ENABLE_DOC)
    include(${CMAKE_SOURCE_DIR}/thirdparty/doxygen.cmake)
endif()

include(${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/opencv.cmake)
include(${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/tesseract.cmake)

set(GCC_FLAGS
    -Werror -Wall -Wduplicated-cond -Wformat=2 -Weffc++
    -Wdouble-promotion -Wuseless-cast -Wnull-dereference
    -Wlogical-op -Wduplicated-branches -Wmisleading-indentation
    -Wsign-conversion -Wpedantic -Wconversion -Woverloaded-virtual
    -Wunused -Wextra -Wshadow -Wnon-virtual-dtor -pedantic -Wold-style-cast
    -Wcast-align -O3
)

add_executable(${PROJECT_NAME}
    ${CMAKE_SOURCE_DIR}/src/main.cc
)
target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_SOURCE_DIR}/include
    ${OpenCV_INCLUDE_DIRS}
)
target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBS})
set_target_properties(${PROJECT_NAME}
    PROPERTIES
    ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
    LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin
    CXX_STANDARD 17
    CXX_EXTENSIONS OFF
    CXX_STANDARD_REQUIRED ON
)
if(CMAKE_BUILD_TYPE STREQUAL "Release")
    if(MSVC)
    else()
        target_compile_options(${PROJECT_NAME} PRIVATE ${GCC_FLAGS})
    endif()
else()
    if(MSVC)
    else()
        list(APPEND GCC_FLAGS -g3 -DNDEBUG)
        target_compile_options(${PROJECT_NAME} PRIVATE ${GCC_FLAGS})
    endif()
endif()

include(${CMAKE_SOURCE_DIR}/cmake/CPackConfig.cmake)