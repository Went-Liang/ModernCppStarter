[![Actions Status](https://github.com/TheLartians/ModernCppStarter/workflows/MacOS/badge.svg)](https://github.com/TheLartians/ModernCppStarter/actions)
[![Actions Status](https://github.com/TheLartians/ModernCppStarter/workflows/Windows/badge.svg)](https://github.com/TheLartians/ModernCppStarter/actions)
[![Actions Status](https://github.com/TheLartians/ModernCppStarter/workflows/Ubuntu/badge.svg)](https://github.com/TheLartians/ModernCppStarter/actions)
[![Actions Status](https://github.com/TheLartians/ModernCppStarter/workflows/Style/badge.svg)](https://github.com/TheLartians/ModernCppStarter/actions)
[![Actions Status](https://github.com/TheLartians/ModernCppStarter/workflows/Install/badge.svg)](https://github.com/TheLartians/ModernCppStarter/actions)
[![codecov](https://codecov.io/gh/TheLartians/ModernCppStarter/branch/master/graph/badge.svg)](https://codecov.io/gh/TheLartians/ModernCppStarter)

<p align="center">
  <img src="https://repository-images.githubusercontent.com/254842585/4dfa7580-7ffb-11ea-99d0-46b8fe2f4170" height="175" width="auto" />
</p>

[英文版](README.md)

# ModernCppStarter

创建一个新的C++项目通常需要大量的准备工作和模板代码，尤其是对于有测试、可执行文件和持续集成的现代C++项目而言。
这个模板是从多个以前的项目中获得的经验总结，应该有助于减少设置一个现代C++项目所需的工作量。

## Features

- [现代 CMake 实践](https://pabloariasal.github.io/2018/02/19/its-time-to-do-cmake-right/)
- 适用于任何规模项目，无论单头文件库还是大型项目
- 库和可执行代码的干净分离
- 集成测试套件：包含了一组用于测试代码功能的测试用例，确保代码的质量和稳定性
- 通过[GitHub Actions](https://help.github.com/en/actions/)实现持续集成CI：使用GitHub的自动化工具，每次代码更改时都会触发一系列的测试、构建和部署任务
- 通过[codecov](https://codecov.io)进行代码覆盖率分析：分析测试套件覆盖了多少代码，帮助开发者找出未被测试覆盖的代码部分
- 通过 [Format.cmake](https://github.com/TheLartians/Format.cmake)强制使用[clang-format](https://clang.llvm.org/docs/ClangFormat.html)和 [cmake-format](https://github.com/cheshirekow/cmake_format)规范代码格式 
- 通过 [CPM.cmake](https://github.com/TheLartians/CPM.cmake)实现可重复的依赖管理：管理项目的依赖，确保在不同环境或不同时间构建项目时，依赖的版本都是一致的
  >在CMake中，CPM是一个基于CMake的优秀依赖管理器，特别适用于C++项目。它是一个轻量级的依赖管理工具，设计目标是简单、高效。CPM不需要任何外部依赖项，可以直接集成到CMake项目中。只要电脑连接了互联网，任何CMake C++项目都可借助CPM添加外部依赖库。



- 通过[PackageProject.cmake](https://github.com/TheLartians/PackageProject.cmake)生成可安装的目标，并自动包含版本信息和头文件
- 使用 [Doxygen](https://www.doxygen.nl)  和 [GitHub Pages](https://pages.github.com) 自动实现[documentation](https://thelartians.github.io/ModernCppStarter)和部署：Doxygen是一个用于从源代码中生成文档的工具，而GitHub Pages则是一个用于托管静态网页的平台。结合两者，可以自动地从代码生成文档，并部署到网页上供人查阅
- 支持 [sanitizer tools等](#additional-tools)：sanitizer工具通常用于内存错误检测，如地址越界、内存泄漏等。这个功能意味着该工具或框架集成了对这类工具的支持，以帮助开发者更好地检测代码中的潜在问题

## 使用

### 按需调整模版

- 使用这个仓库 [as a template](https://help.github.com/en/github/creating-cloning-and-archiving-repositories/creating-a-repository-from-a-template).
- 在相关的CMakeLists.txt文件中，将所有出现的“Greeter”替换为您的项目名称
  - 这里大小写很重要：Greeter表示项目名称，而greeter用于文件名
  - 记得将include/greeter目录重命名为您的项目的小写名称，并相应地更新所有相关的#include指令
- 用您自己的源文件替换它们  
- 对于仅包含头文件的库：请查[CMakeLists.txt](CMakeLists.txt)中的注释 
- 将您项目的[codecov令牌](https://docs.codecov.io/docs/quick-start)添加到您的项目的github密钥下，键名为`CODECOV_TOKEN`

最后，您可以删除任何未使用的文件，例如独立的目录或与您的项目无关的github工作流程。请随意将许可证替换为适合您项目的许可证。

为了清晰地分离库和子项目代码，外部的CMakeList.txt文件仅定义了库本身，而测试和其他子项目则包含在其自己的目录中。在开发过程中，通常方便的是[一次性构建所有子项目](#一次性构建所有内容)。

### 构建并运行独立目标

使用以下命令构建并运行可执行目标

```bash
# 从 standalone 目录读取源代码和CMakeLists.txt 文件，
# 并将生成的构建系统（如 Makefile 或 Visual Studio 项目文件）输出到 build/standalone 录
cmake -S standalone -B build/standalone 
# 在 build/standalone 目录中执行构建过程
cmake --build build/standalone
./build/standalone/Greeter --help
```

### 构建并运行测试套件

从项目的根目录使用以下命令来运行测试套件

```bash
cmake -S test -B build/test
cmake --build build/test

# 在 build/test 目录下构建项目，
# 并运行所有定义的测试（通过 test 目标）。
# 如果任何测试失败，由于 CTEST_OUTPUT_ON_FAILURE=1 的设置，CTest 将显示这些失败测试的详细输出。
CTEST_OUTPUT_ON_FAILURE=1 cmake --build build/test --target test

# or simply call the executable: 
./build/test/GreeterTests
```

要收集代码覆盖率信息，请使用`-DENABLE_TEST_COVERAGE=1`选项运行 CMake。

### 运行 clang-format

从项目的根目录使用以下命令来检查和修复 C++ 和 CMake 源代码样式。
这需要在当前系统上安装 _clang-format_、_cmake-format_ 和 _pyyaml_。

```bash
cmake -S test -B build/test

# view changes
cmake --build build/test --target format

# apply changes
cmake --build build/test --target fix-format
```

查看 [Format.cmake](https://github.com/TheLartians/Format.cmake) 细节

可以用pip安装上述依赖：
```bash
pip install clang-format==14.0.6 cmake_format==0.6.11 pyyaml
```

### 构建文档

每当创建 [GitHub Release](https://help.github.com/en/github/administering-a-repository/managing-releases-in-a-repository) 时，都会自动构建和[发布](https://thelartians.github.io/ModernCppStarter)文档。
要手动构建文档，请调用以下命令。

```bash
cmake -S documentation -B build/doc
cmake --build build/doc --target GenerateDocs
# view the docs
open build/doc/doxygen/html/index.html
```

要在本地构建文档，您需要在系统上安装 Doxygen、jinja2 和 Pygments。

### 一次性构建所有内容

该项目还包括一个“all”目录，允许同时构建所有目标。
这在开发过程中非常有用，因为它将所有子项目公开给您的 IDE，并避免了库的冗余构建。

```bash
cmake -S all -B build
cmake --build build

# run tests
./build/test/GreeterTests
# format code
cmake --build build --target fix-format
# run standalone
./build/standalone/Greeter --help
# build docs
cmake --build build --target GenerateDocs
```

### 附加工具
测试和独立的子项目包含了[tools.cmake](cmake/tools.cmake) 文件，该文件用于通过CMake配置参数按需导入额外的工具。

目前支持以下功能：

#### Sanitizers

可以通过配置 CMake 来启用Sanitizers`-DUSE_SANITIZER=<Address | Memory | MemoryWithOrigins | Undefined | Thread | Leak | 'Address;Undefined'>`.

#### Static Analyzers

静态分析器可以通过设置 -DUSE_STATIC_ANALYZER=<clang-tidy | iwyu | cppcheck> 来启用，或者将这些选项用引号括起来并用分号分隔，以组合使用多个分析器。

默认情况下，分析器会自动查找如 .clang-format 这样的配置文件。

可以通过设置 CLANG_TIDY_ARGS、IWYU_ARGS 或 CPPCHECK_ARGS 变量来向分析器传递额外的参数。

#### Ccache

可以通过配置 CMake 来启用Ccache`-DUSE_CCACHE=<ON | OFF>`.

>Ccache是一个编译器缓存。它会在编译器编译时存储信息，以供下一次编译时使用，从而大大加快编译速度。Ccache以空间换取速度，特别适合经常进行“make clean”操作或删除输出目录后重新编译的情况。通过使用Ccache，无论是普通编译还是大型项目的编译，都可以显著提高编译速度，通常可以提高5到10倍。

## FAQ

> 我可以将其用于header-only库吗？

是的，但是您需要将库类型更改为 INTERFACE 库，如[CMakeLists.txt](CMakeLists.txt)中所述。

请参阅[此处](https://github.com/TheLartians/StaticTypeInfo)，了解基于模板的header-only库示例。

> 我不需要standalone target / documentation。我怎样才能摆脱它？

只需删除 standalone / documentation目录并根据 github 工作流程文件即可。

> 我可以同时构建独立的子项目和测试吗？/ 我如何告诉我的集成开发环境（IDE）所有子项目的信息？

为了保持模板的模块化，所有从库中派生的子项目都被分隔到它们自己的CMake模块中。
这种做法使得第三方项目重用项目库代码变得轻而易举。
为了让集成开发环境（IDE）能够看到项目的全部范围，模板中包含了all目录，该目录将为所有子项目创建一个单独的构建。
为了获得最佳的IDE支持，请将此目录作为主目录使用。

> 我看到您正在使用“GLOB”在 CMakeLists.txt 中添加源文件。 这不是evil邪恶的吗？

Glob 被认为是不好的，因为对源文件结构的任何更改[可能不会被 CMake 的构建器自动捕获](https://cmake.org/cmake/help/latest/command/file.html#filesystem)，并且您需要在更改时手动调用 CMake。

我个人更喜欢 GLOB 解决方案，因为它简单，且可以随意将其更改为显式列出源。

> 我想创建依赖于我的库的其他目标。我应该修改主 CMakeLists 以包含它们吗？

避免在库的CMakeLists中包含派生项目（尽管这在C++世界中很常见），因为这实际上会颠倒依赖树，使构建系统难以理解。

相反，创建一个包含CMakeLists的新目录或项目，并将库作为依赖项添加（例如，像[standalone目录](standalone/CMakeLists.txt)那样）。

根据类型，将这些组件移到单独的存储库中，并引用库的特定提交或版本，这样做可能更有意义。

这样做的优点在于，可以独立地改进和更新各个库和组件。

> 您建议使用 CPM.cmake 添加外部依赖项。 这会迫使我的库的用户也使用 CPM.cmake 吗？

[CPM.cmake](https://github.com/TheLartians/CPM.cmake) 应该对库用户来说是透明的，因为它是一个独立的 CMake 脚本。
如果出现问题，用户可以通过定义 CMake 或环境变量 [`CPM_USE_LOCAL_PACKAGES`](https://github.com/cpm-cmake/CPM.cmake#options) 来选择退出，这将用相应的 find_package 调用覆盖所有对 CPMAddPackage 的调用。
这也应该允许用户使用他们最喜欢的外部 C++ 依赖管理器（如 vcpkg 或 Conan）与项目一起使用。

> 我可以在离线状态下配置和构建我的项目吗？

构建项目不需要互联网连接，但在使用 CPM 时，缺少的依赖项会在配置时下载。

为了避免冗余的下载，强烈建议设置一个 CPM.cmake 缓存目录，例如：export CPM_SOURCE_CACHE=$HOME/.cache/CPM。
这将启用浅克隆，并允许在缓存中已存在依赖项时进行离线配置。

> 我可以使用 CPack 为我的项目创建包安装程序吗？

由于存在很多可能的选项和配置，这（还）不在此模板的范围内。有关设置 CPack 安装程序的更多信息，请参阅 [CPack documentation](https://cmake.org/cmake/help/latest/module/CPack.html) 。

> 这太复杂了，我只想玩一下 C++ 代码并测试一些库

也许 [MiniCppStarter](https://github.com/TheLartians/MiniCppStarter) 适合你！

## 相关项目和替代方案

- [**ModernCppStarter & PVS-Studio Static Code Analyzer**](https://github.com/viva64/pvs-studio-cmake-examples/tree/master/modern-cpp-starter): 有关如何将 ModernCppStarter 与 PVS-Studio 静态代码分析器结合使用的官方说明。
- [**cpp-best-practices/gui_starter_template**](https://github.com/cpp-best-practices/gui_starter_template/): 一个流行的 C++ 入门项目，创建于 2017 年。
- [**filipdutescu/modern-cpp-template**](https://github.com/filipdutescu/modern-cpp-template):最近的初学者使用更传统的 CMake 结构和依赖管理方法。
- [**vector-of-bool/pitchfork**](https://github.com/vector-of-bool/pitchfork/): Pitchfork 是一组 C++ 项目约定。

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=TheLartians/ModernCppStarter,cpp-best-practices/gui_starter_template,filipdutescu/modern-cpp-template&type=Date)](https://star-history.com/#TheLartians/ModernCppStarter&cpp-best-practices/gui_starter_template&filipdutescu/modern-cpp-template&Date)
