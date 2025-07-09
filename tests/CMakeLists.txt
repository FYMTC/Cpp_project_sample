cmake_minimum_required(VERSION 3.16.3)
project(test)
# 将 gtest 添加到项目中
# 关闭 gmock 功能
set(BUILD_GMOCK OFF)
add_subdirectory(${HELLO_WORLD_PROJECT_ROOT}/thirdpart/googletest-1.13.0 googletest)

add_executable(test_hello ${CMAKE_CURRENT_SOURCE_DIR}/test_hello.cpp)

target_include_directories(test_hello PUBLIC ${HELLO_WORLD_PROJECT_ROOT})

target_link_libraries(test_hello PUBLIC hellolib GTest::gtest GTest::gtest_main)

# 自动复制依赖 DLL 到 test_hello 可执行文件目录
add_custom_command(TARGET test_hello POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        $<TARGET_FILE:hellolib>
        $<TARGET_FILE_DIR:test_hello>
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        $<TARGET_FILE:worldlib>
        $<TARGET_FILE_DIR:test_hello>
)

add_test(NAME test_hello
         COMMAND test_hello)
