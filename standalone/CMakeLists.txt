cmake_minimum_required(VERSION 3.14...3.22)

project(GreeterStandalone LANGUAGES CXX)

# --- Import tools ----

include(../cmake/tools.cmake)

# ---- Dependencies ----

include(../cmake/CPM.cmake)

# 使用CPM来添加一个名为cxxopts的包。它指定了包的GitHub仓库地址、版本号以及一些选项。
# 这些选项告诉CPM在安装cxxopts时不构建示例和测试，但要启用安装步骤
CPMAddPackage(
  GITHUB_REPOSITORY jarro2783/cxxopts
  VERSION 3.0.0
  OPTIONS "CXXOPTS_BUILD_EXAMPLES NO" "CXXOPTS_BUILD_TESTS NO" "CXXOPTS_ENABLE_INSTALL YES"
)

# 使用CPM来添加一个名为Greeter的包，这次它是一个本地包，源代码目录位于当前CMakeLists.txt文件的上一级目录
CPMAddPackage(NAME Greeter SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/..)

# ---- Create standalone executable ----

# 这行代码使用GLOB命令来查找当前目录下的source子目录中所有的.cpp文件，并将它们存储在一个名为sources的变量中。
# CONFIGURE_DEPENDS选项意味着当这些文件改变时，CMake会重新运行以重新配置项目
file(GLOB sources CONFIGURE_DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/source/*.cpp)

# 这行代码告诉CMake创建一个可执行文件，名称为PROJECT_NAME变量的值（在这个例子中是GreeterStandalone），并使用之前找到的所有.cpp源文件
add_executable(${PROJECT_NAME} ${sources})

# 这行代码设置了可执行文件的目标属性，指定了C++标准为C++17，并将输出的可执行文件名设置为"Greeter"
set_target_properties(${PROJECT_NAME} PROPERTIES CXX_STANDARD 17 OUTPUT_NAME "Greeter")

# 这行代码链接了之前添加的Greeter库和cxxopts库到可执行文件。这意味着这些库中的函数和对象可以在GreeterStandalone程序中使用
target_link_libraries(${PROJECT_NAME} Greeter::Greeter cxxopts)
