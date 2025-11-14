
cmake_minimum_required(VERSION 3.16.3)
project(hello_world)

enable_testing()

option(BUILD_DEBUG "Builds debug!!!" OFF)
if(BUILD_DEBUG)
    add_compile_options("-g")
endif()

option(ENABLE_COVERAGE "Enable gcov/gcovr coverage instrumentation" OFF)
if(ENABLE_COVERAGE)
    if(NOT BUILD_TEST)
        message(FATAL_ERROR "ENABLE_COVERAGE requires BUILD_TEST=ON so tests can run.")
    endif()
    if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
        set(COVERAGE_COMPILE_FLAGS -O0 -g --coverage)
        set(COVERAGE_LINK_FLAGS --coverage)
        add_compile_options(${COVERAGE_COMPILE_FLAGS})
        add_link_options(${COVERAGE_LINK_FLAGS})
    else()
        message(FATAL_ERROR "Coverage is currently supported only with GCC or Clang toolchains.")
    endif()

    find_program(GCOVR_PATH gcovr)
    if(NOT GCOVR_PATH)
        message(WARNING "gcovr executable not found; coverage reports will not be generated.")
    endif()
endif()

set(HELLO_WORLD_PROJECT_ROOT ${CMAKE_CURRENT_SOURCE_DIR})
set(HELLO_WORLD_BIN_ROOT ${CMAKE_CURRENT_BINARY_DIR})

add_subdirectory(hello)
add_subdirectory(world)

add_executable(main ${CMAKE_CURRENT_SOURCE_DIR}/main.cpp)

target_link_libraries(main PUBLIC hellolib)
target_link_libraries(main PUBLIC worldlib)

# 设置 编译测试选项
option(BUILD_TEST "Builds test!!!" ON)
if(BUILD_TEST)
    add_subdirectory(tests)
endif()

if(ENABLE_COVERAGE AND BUILD_TEST AND GCOVR_PATH)
    set(COVERAGE_OUTPUT_DIR ${HELLO_WORLD_BIN_ROOT}/coverage)
    add_custom_target(coverage
        COMMAND ${CMAKE_COMMAND} -E make_directory ${COVERAGE_OUTPUT_DIR}
        COMMAND ${CMAKE_CTEST_COMMAND} --output-on-failure
        COMMAND ${GCOVR_PATH}
            --root ${HELLO_WORLD_PROJECT_ROOT}
            --html-details ${COVERAGE_OUTPUT_DIR}/index.html
            --xml ${COVERAGE_OUTPUT_DIR}/coverage.xml
            --txt ${COVERAGE_OUTPUT_DIR}/summary.txt
            --lcov ${COVERAGE_OUTPUT_DIR}/lcov.info
            --exclude ${HELLO_WORLD_PROJECT_ROOT}/thirdpart
            --exclude ${HELLO_WORLD_PROJECT_ROOT}/tests/googletest
            --print-summary
            --fail-under-branch 0
        WORKING_DIRECTORY ${HELLO_WORLD_BIN_ROOT}
        COMMENT "Running tests and generating coverage report"
    )
    if(TARGET test_hello)
        add_dependencies(coverage test_hello)
    endif()
endif()
