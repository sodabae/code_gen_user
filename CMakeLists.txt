cmake_minimum_required(VERSION 3.16)

project(PrintApp
    VERSION 1.0
    LANGUAGES CXX
)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

#-----------------------
# Build generator from code_gen Repo
#-----------------------
include(ExternalProject)
include(ProcessorCount)
ExternalProject_Add(
    generator_project
    GIT_REPOSITORY https://github.com/sodabae/code_gen.git
    GIT_TAG master
    PREFIX ${CMAKE_BINARY_DIR}/generator_project
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
    BUILD_COMMAND ${CMAKE_COMMAND} --build <BINARY_DIR> -- -j${PocessorCount}
    INSTALL_COMMAND ""   #Don't do an install of this
    LOG_DOWNLOAD ON
    LOG_BUILD ON
)
#path to generator executable after build
set(GENERATOR_EXE ${CMAKE_BINARY_DIR}/generator_project/src/generator_project-build/generator)

#-----------------------
# Create the Generated file (print.cpp)
#-----------------------

set(FILE_NAME print.cpp)
set(GENERATED_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)
set(GENERATED_FILE ${GENERATED_DIR}/${FILE_NAME})

file(MAKE_DIRECTORY ${GENERATED_DIR})

#Allow the user to configure the type
set(PRINT_TYPE 1 CACHE STRING "Type passed to generator")

#Note: "generator" must be the add_executable target in the code_gen repo CMakeLists.txt
add_custom_command(
    OUTPUT ${GENERATED_FILE}
    COMMAND ${GENERATOR_EXE}
        --type=${PRINT_TYPE}
        --path=${GENERATED_DIR}
        --file=${FILE_NAME}
    DEPENDS ${GENERATOR_EXE}
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
    COMMENT "Running external generator to create print.cpp"
    VERBATIM
)

#-----------------------
# Build main app
#-----------------------
add_custom_target(generate_print ALL DEPENDS ${GENERATED_FILE})

add_executable(print_app
    src/main.cpp
    ${GENERATED_FILE}
)

add_dependencies(print_app generate_print)

target_include_directories(print_app
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/include
)

target_compile_options(print_app PRIVATE
    -Wall
    -Wextra
    -Wpedantic
)
