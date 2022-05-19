# bazel
Bazel 是一款类似于 Make、Maven 和 Gradle的开源构建和测试工具。它使用可读的高级构建语言，支持多种变成语言编写的项目，并且能够为多个平台进行构建。Bazel 支持构建包含多个仓库、大量开发人员的大型代码库。

## 为什么需要构建工具
对于一个简单的项目 (以C++项目为例) 来说，可能只包含简单的几个头文件 (.h文件) 和源文件 (.cc文件)，这时我们可以将这些文件放到一个目录下，然后使用一条简单的命令完成项目构建：

```g++ *.cc```
这里g++工具会帮助我们依次调用预处理器、编译器、汇编器、链接器，构建出目标文件。因此，对于小型项目来说，一个g++编译器就能够一次性完成项目构建；但对于大型项目，考虑以下几种情况：

- 项目的不同部分通常由不同的编程语言编写
- 项目通常存在外部依赖，这些依赖代码位于本地不同位置或远程代码仓库
- 项目通常构建耗时，希望构建时仅对更改的部分重建
很显然，在上述情况下，编译器不能满足我们的需求。这时，构建工具的作用就显现出来了：构建工具通过解析我们制定的编译规则，自动配置依赖环境，调用编译器，完成项目构建。我们依然可以通过一条命令完成整个项目的构建。

Bazel正是一款帮助我们实现自动化构建的工具：Bazel能够缓存所有以前完成的工作，并跟踪对文件内容和构建命令的更改，因此Bazel在构建时只对需要重建的部分进行构建；同时，Bazel支持项目以高度并行和增量的方式构建，能够进一步加快构建速度。


## 构建C++项目
### 了解工作空间
在构建项目之前，您需要先设置其工作空间。工作空间是一个目录，用于保存项目的源文件和 Bazel 的构建输出。它还包含 Bazel 识别为特殊的文件：

- WORKSPACE 文件，用于将目录及其内容标识为 Bazel 工作区，它位于项目的目录结构的根目录下。

- 一个或多个 BUILD 文件，用于告知 Bazel 如何构建项目的不同部分。

### 了解BUILD文件
BUILD 文件包含几种不同类型的 Bazel 指令。最重要的类型是构建规则，告知 Bazel 如何构建所需的输出，例如可执行二进制文件或库。
工作区中包含 BUILD 文件的目录称为包 (package).

### 仓库 (Repositories)
代码被组织在仓库中。包含 WORKSPACE 文件的目录为仓库的根目录，称为 @；这个名称也可以在 WORKSPACE 文件中使用 workspace 规则进行定义，例如以下代码将当前仓库名称定义为 @my_repo：
```yaml
workspace(name = "my_repo")
```

### 包 (Packages)
包是仓库中代码的基础单元，包是有关的文件和其依赖清单的集合。包被定义在一个包含 BUILD 或 BUILD.bazel 文件的目录下，其包含目录下的所有文件以及子目录。 
包中其他包含 BUILD 文件的目录会被排除在当前包之外。

example中的 data_clean 目录中包含 BUILD 文件，因此 data_clean 目录下的所有文件和子目录构成了一个包。

### 目标 (Targets)
包中的元素被称为目标。目标通常有2种类型：文件 (files) 和规则 (rules)。 文件包括开发人员编写的源文件 (source files) 和构建工具根据规则从源文件生成的生成文件 (generated files / derived files / output files)；
规则说明了输入和输出的关系或从输入生成输出的步骤。

example中的 `data_clean/BUILD` 文件中通过 `wasm_cc_binary` 制定了一条从源文件 (输入) `plugin.cc、plugin.h` 到可执行程序 (输出) `data-clean.wasm` 的规则。

### 标签 (Labels)
目标的名字被称为标签。标签通常具有以下形式：
```yaml
@myrepo//my/app/main:app_binary
```
第一部分 @myrepo// 为仓库的名称。当前仓库的可以简写为 //，项目主仓库名称为 @//。如果当前项目不是其他项目的外部依赖，当前仓库就是主仓库。

第二部分 my/app/main 为包相对根目录的位置，前两部分组合 @myrepo//my/app/main 构成了完整的包名。

第三部分 app_binary 为目标，这个目标可以是包中的文件，或者是定义在 BUILD 文件中的规则。

如 
```shell
@proxy_wasm_cpp_sdk//bazel/wasm:wasm.bz
```
它的文件地址就是
```shell
https://github.com/proxy-wasm/proxy-wasm-cpp-sdk/blob/master/bazel/wasm/wasm.bzl
```

标签的简写规则：
- 当前仓库可以简写为 //
- 当前包中可以省略包名以及冒号，例如在 @myrepo//my/app/main 包中可以将 @myrepo//my/app/main:app_binary 省略简写为 app_binary 或 :app_binary
- 目标名称和其目录名称一致时可以省略目标名称以及冒号，例如可以将 //my/app/lib:lib 简写为 //my/app/lib
案例中 BUILD 文件中 load 函数的第一个参数就是一个标签：
```shell
load("@proxy_wasm_cpp_sdk//bazel/wasm:wasm.bzl", "wasm_cc_binary")
```
标签`@proxy_wasm_cpp_sdk//bazel/wasm:wasm.bzl`,仓库名称为`@proxy_wasm_cpp_sdk`,包的位置为bazel目录下的wasm目录，目标为wasm.bzl文件。

在构建项目时也用到了标签：
```shell
bazel build //:data-clean.wasm
```
这里目标`data-clean.wasm`是写在`BUILD`文件中的一条规则。

### 构建规则基础
#### cc_binary
规则 `cc_library` 用来构建可执行文件。

#### cc_library
规则 `cc_library` 用来构建库。

### 创建宏与规则
- 宏：实例化规则的函数。当我们需要在 BUILD 文件中重复使用一些功能时可以定义宏。
- 规则：规则定义了 Bazel 从输入构建输出的一系列动作，规则比宏的功能更加强大，能够控制 Bazel 内部的整个构建执行流程。


### 示例：
```yaml
load("@proxy_wasm_cpp_sdk//bazel/wasm:wasm.bzl", "wasm_cc_binary")

wasm_cc_binary(
name = "data-clean.wasm",
srcs = [
"plugin.cc",
"plugin.h",
],
deps = [
"@proxy_wasm_cpp_sdk//:proxy_wasm_intrinsics",
],
)
```
BUILD文件首先通过`load`函数将`@proxy_wasm_cpp_sdk//bazel/wasm:wasm.bzl`文件中的`wasm_cc_binary`函数导入，该函数用以定义构建`C++ Wasm`可执行文件的规则。
而后调用 `wasm_cc_binary` 函数定义构建目标 (target) `data-clean.wasm` 的规则，其中参数 `srcs` 指定了源文件,`name`指定了目标名称，构建时需要对应,`deps`指定了依赖文件。

运行以下命令以构建 data-clean.wasm：

```shell 
bazel build //:data-clean.wasm
```
构建成功后在项目的根目录下产生了 bazel-bin bazel-out bazel-stage1 bazel-testlogs 这4个文件夹 (特别注意这些文件夹是软链接，其实际位置可通过命令 ls -l 查找)，
在 bazel-bin/main 中生成了目标可执行文件 data-clean.wasm。

### 参考
[Bazel 学习笔记 (一) 快速开始](https://zhuanlan.zhihu.com/p/411563404)

[Bazel 学习笔记 (二) C++ 基础构建规则](https://zhuanlan.zhihu.com/p/412355447)

[Bazel 学习笔记 (三) 依赖外部仓库](https://zhuanlan.zhihu.com/p/415586721)

[Bazel 学习笔记 (四) 创建宏与规则](https://zhuanlan.zhihu.com/p/421489117)



