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
#  In the CMake config phase, need to 
#  1. clone
#  2. config
#  3. build
#  4. generate source
#-----------------------
set(GENERATOR_DIR ${CMAKE_BINARY_DIR}/generator_project)

# Clone repo if it doesn't exist
if(NOT EXISTS ${GENERATOR_DIR}/CMakeLists.txt)
    message(STATUS "Cloning generator repo...")
    execute_process(
        COMMAND git clone https://github.com/sodabae/code_gen.git ${GENERATOR_DIR}
        RESULT_VARIABLE GIT_CLONE_RESULT
    )
    if(NOT GIT_CLONE_RESULT EQUAL 0)
        message(FATAL_ERROR "Failed to clone generator repo")
    endif()
endif()



#path to generator executable after build
set(GENERATOR_EXE ${GENERATOR_DIR}/build/generator)

#...................................
# 2. config

execute_process(
    COMMAND ${CMAKE_COMMAND} -S ${GENERATOR_DIR} -B ${GENERATOR_DIR}/build
    RESULT_VARIABLE config_result
)

if(NOT config_result EQUAL 0)
    message(FATAL_ERROR "Failed to config the generator")
endif()

#...................................
# 3. build

execute_process(
    COMMAND ${CMAKE_COMMAND} --build ${GENERATOR_DIR}/build
    RESULT_VARIABLE build_result
)

if(NOT build_result EQUAL 0)
    message(FATAL_ERROR "Failed to build the generator")
endif()

#...................................
# 4. generate source code

set(FILE_NAME print.cpp)
set(GENERATED_DIR ${CMAKE_BINARY_DIR}/generated)
set(GENERATED_FILE ${GENERATED_DIR}/${FILE_NAME})

file(MAKE_DIRECTORY ${GENERATED_DIR})

#Allow the user to configure the type
set(PRINT_TYPE 1 CACHE STRING "Type passed to generator")

execute_process (
    COMMAND ${GENERATOR_EXE}
        --type=${PRINT_TYPE}
        --path=${GENERATED_DIR}
        --file=${FILE_NAME}
    RESULT_VARIABLE gen_result
)

if (NOT gen_result EQUAL 0)
    message(FATAL_ERROR "Unable to generate src files")
endif()


#-----------------------
# Build main app
#-----------------------

file(GLOB SRC_FILES
     "${GENERATED_DIR}/*.cpp"
     )
add_executable(print_app
    src/main.cpp
    ${SRC_FILES}
)

target_include_directories(print_app
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/include
)

target_compile_options(print_app PRIVATE
    -Wall
    -Wextra
    -Wpedantic
)
