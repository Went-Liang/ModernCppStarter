cmake_minimum_required(VERSION 3.14...3.22)

# ---- Project ----

# Note: update this to your new project's name and version
project(
  Greeter
  VERSION 1.0
  LANGUAGES CXX
)

# ---- Include guards ----
# 检查项目的源代码目录（PROJECT_SOURCE_DIR）和构建目录（PROJECT_BINARY_DIR）是否相同。如果是相同的，那么它会输出一个致命错误消息，并终止CMake的配置过程。
if(PROJECT_SOURCE_DIR STREQUAL PROJECT_BINARY_DIR)
  message(
    FATAL_ERROR #FATAL_ERROR：这是message()函数的一个参数，它告诉CMake这是一个致命错误，应该立即停止配置过程
      "In-source builds not allowed. Please make a new directory (called a build directory) and run CMake from there."
  )
endif()

# ---- Add dependencies via CPM ----
# see https://github.com/TheLartians/CPM.cmake for more info
# 加载CPM.cmake文件，CPM是一个CMake的包管理器，用于添加和管理第三方库
include(cmake/CPM.cmake)

# 使用CPM添加依赖包：这里添加了PackageProject.cmake，它用于使目标可安装
CPMAddPackage("gh:TheLartians/PackageProject.cmake@1.8.0")
# 使用CPM添加一个名为fmt的依赖包，指定了Git标签（版本号）和GitHub仓库地址，并设置了选项以创建一个可安装的目标
CPMAddPackage(
  NAME fmt
  GIT_TAG 9.1.0
  GITHUB_REPOSITORY fmtlib/fmt
  OPTIONS "FMT_INSTALL YES" # create an installable target
)

# ---- Add source files ----

# Note: GLOB被认为是不好的做法，因为 CMake 的生成器可能无法检测到新文件
# 使用file(GLOB_RECURSE ...)命令递归地查找项目中的头文件和源文件
# CONFIGURE_DEPENDS表示这些文件在配置阶段会被检查，如果文件有变化，CMake会重新运行。
file(GLOB_RECURSE headers CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/include/*.h")
file(GLOB_RECURSE sources CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/source/*.cpp")

# ---- Create library ----

# Note: 对于仅标头库，将所有 PUBLIC 标志更改为 INTERFACE 并创建一个接口
# target: add_library(${PROJECT_NAME} INTERFACE)
# 创建一个库目标，库的名称是项目名称（Greeter），并添加了之前找到的头文件和源文件
add_library(${PROJECT_NAME} ${headers} ${sources})
# 设置目标的C++标准为C++17
set_target_properties(${PROJECT_NAME} PROPERTIES CXX_STANDARD 17)

# 为项目设置编译选项，这里指定了MSVC编译器的选项，以确保跨平台的一致性
target_compile_options(${PROJECT_NAME} PUBLIC "$<$<COMPILE_LANG_AND_ID:CXX,MSVC>:/permissive->")

# 链接fmt::fmt库到项目中，fmt是一个用于C++的格式化库
target_link_libraries(${PROJECT_NAME} PRIVATE fmt::fmt)

# 设置目标的包含目录(包括build和install的目录)，这样编译器在编译时能够找到依赖的头文件
target_include_directories(
  ${PROJECT_NAME} PUBLIC $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
                         $<INSTALL_INTERFACE:include/${PROJECT_NAME}-${PROJECT_VERSION}>
)

# ---- Create an installable target ----
# 这允许用户通过“find_package()”安装和查找库.

# 项目版本header的放置位置应与项目的常规版本header相匹配
# 创建一个变量VERSION_HEADER_LOCATION，它包含了项目版本头文件的路径，这个路径是小写的项目名称后跟/version.h
string(TOLOWER ${PROJECT_NAME}/version.h VERSION_HEADER_LOCATION)

# 定义了如何打包和安装项目，包括名称、版本、命名空间、二进制目录、包含目录、安装目的地、版本头文件和兼容性信息。
packageProject(
  NAME ${PROJECT_NAME}
  VERSION ${PROJECT_VERSION}
  NAMESPACE ${PROJECT_NAME}
  BINARY_DIR ${PROJECT_BINARY_DIR}
  INCLUDE_DIR ${PROJECT_SOURCE_DIR}/include
  INCLUDE_DESTINATION include/${PROJECT_NAME}-${PROJECT_VERSION}
  VERSION_HEADER "${VERSION_HEADER_LOCATION}"
  COMPATIBILITY SameMajorVersion #兼容性
  DEPENDENCIES "fmt 9.1.0"
)
