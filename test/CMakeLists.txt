cmake_minimum_required(VERSION 3.14...3.22)

project(GreeterTests LANGUAGES CXX)

# ---- Options ----
# 定义两个选项，可以通过CMake的命令行参数或图形界面来设置
# 是否启用测试覆盖率
option(ENABLE_TEST_COVERAGE "Enable test coverage" OFF)
# 是否测试通过find_package找到的版本
option(TEST_INSTALLED_VERSION "Test the version found by find_package" OFF)

# --- Import tools ----
# 包含一些自定义的工具或函数
include(../cmake/tools.cmake)

# ---- Dependencies ----
# CMake的包管理器，用于添加和管理第三方库
include(../cmake/CPM.cmake)
# 使用CPM添加一个名为doctest的包，版本是2.4.9，从GitHub获取
CPMAddPackage("gh:doctest/doctest@2.4.9")
# 添加一个名为Format.cmake的包，版本是1.7.3，同样从GitHub获取。
CPMAddPackage("gh:TheLartians/Format.cmake@1.7.3")

# 如果TEST_INSTALLED_VERSION选项被设置为ON，则通过find_package查找并使用已安装的Greeter包
if(TEST_INSTALLED_VERSION)
  find_package(Greeter REQUIRED)
else()
  CPMAddPackage(NAME Greeter SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/..)
  # 否则，使用CPM添加一个名为Greeter的包，源代码目录是当前CMakeLists.txt所在的上级目录
endif()

# ---- Create binary ----
# 通过GLOB命令收集当前目录（test）下source子目录中所有的.cpp文件
file(GLOB sources CONFIGURE_DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/source/*.cpp)
# 创建一个可执行文件，名称与项目名称相同，源文件是上面收集的sources列表
add_executable(${PROJECT_NAME} ${sources})
# 链接库文件，这里链接了doctest和Greeter库
target_link_libraries(${PROJECT_NAME} doctest::doctest Greeter::Greeter)
# 设置目标的C++标准为C++17
set_target_properties(${PROJECT_NAME} PROPERTIES CXX_STANDARD 17)

# enable compiler warnings
# 如果不是测试已安装的版本，启用编译器警告
if(NOT TEST_INSTALLED_VERSION) # TEST_INSTALLED_VERSION没有被设置或者其值为假，那么内部的代码块将被执行
# 检查当前使用的C++编译器是否是Clang或GNU（即GCC）。
# CMAKE_CXX_COMPILER_ID是一个CMake变量，用于存储当前C++编译器的ID。MATCHES是一个字符串比较操作符，用于检查一个字符串是否与另一个模式匹配
  if(CMAKE_CXX_COMPILER_ID MATCHES "Clang" OR CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    target_compile_options(Greeter PUBLIC -Wall -Wpedantic -Wextra -Werror)
    # 设置公共的编译选项：打开所有有用的警告、打开对不符合C++标准的代码的警告、打开一些额外的警告、将所有警告转换为错误，即任何警告都会导致编译失败
  elseif(MSVC) # 如果当前编译器是MSVC
    target_compile_options(Greeter PUBLIC /W4 /WX) #设置警告级别为4，这是MSVC编译器中的最高警告级别、将所有警告转换为错误
    target_compile_definitions(${PROJECT_NAME} PUBLIC DOCTEST_CONFIG_USE_STD_HEADERS) # 添加公共的编译定义：用于doctest测试框架，以指示它使用标准库的头文件而不是单文件版本
  endif()
endif()

# ---- Add GreeterTests ----
# 启用测试
enable_testing()

# 注释说明了doctest等测试框架可以自动配置CMake测试，对于其他测试框架，需要添加tests目标
#: add_test(NAME ${PROJECT_NAME} COMMAND ${PROJECT_NAME})

# 包含doctest的CMake脚本，用于发现和添加测试
include(${doctest_SOURCE_DIR}/scripts/cmake/doctest.cmake)
doctest_discover_tests(${PROJECT_NAME})

# ---- code coverage ----
# 如果启用了测试覆盖率，设置编译选项以支持覆盖率工具
if(ENABLE_TEST_COVERAGE)
  target_compile_options(Greeter PUBLIC -O0 -g -fprofile-arcs -ftest-coverage)
  target_link_options(Greeter PUBLIC -fprofile-arcs -ftest-coverage)
endif()
