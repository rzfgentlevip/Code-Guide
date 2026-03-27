---
title: 1、什么是MPC(Make Project Creator)
createTime: 2026/03/26 09:43:09
permalink: /cplus/tools-must/mpc/什么是MPC(Make Project Creator)/
order: 1
icon: streamline-color:lasso-tool
---


## 什么是 MPC (Make Project Creator)

MPC (Make Project Creator) 是一个项目文件生成工具，它通过读取简单的配置文件（.mpc、.mwc等），自动生成各种编译环境所支持的项目文件，如 Visual Studio 的 .sln/.vcxproj、GNU Makefile 等。

> 核心价值：一次编写配置文件，即可生成多种编译工具的项目文件，极大减少了跨平台项目维护的工作量。

## MPC工具集架构

MPC 工具集包含两个主要脚本和四种输入文件类型：

```shell
┌─────────────────────────────────────────────────────────────────┐
│                        输入文件层                                 │
├─────────────────────────────────────────────────────────────────┤
│  .mpc (项目文件)  │  .mwc (工作区文件)  │  .mpb/.mwb (基础文件)   │
└────────┬─────────────────┬──────────────────┬───────────────────┘
         ▼                 ▼                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                        处理脚本层                                 │
├─────────────────────────────────────────────────────────────────┤
│           mpc.pl (项目生成器)  │  mwc.pl (工作区生成器)           │
└─────────────────────────────────────────────────────────────────┘
         │                              │
         ▼                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                        输出文件层                                 │
├─────────────────────────────────────────────────────────────────┤
│  .vcxproj  │  .sln  │  Makefile  │  .dsp  │  其他平台项目文件    │
└─────────────────────────────────────────────────────────────────┘
```


工作流程说明：

- mpc.pl：读取 .mpc 文件，生成单个项目文件

- mwc.pl：读取 .mwc 文件，调用 mpc.pl 生成所有项目，再生成顶层工作区文件

## MPC四种文件类型

| 文件类型   | 全称 / 作用                                                  | 类比                                   |
| :--------- | :----------------------------------------------------------- | :------------------------------------- |
| **`.mpc`** | **项目文件 (Project File)**：定义了一个独立的编译单元（如一个动态库 `.dll`/`.so` 或一个可执行文件 `.exe`）。它包含了该项目具体的源文件列表、头文件路径、依赖库等详细信息。 | **单个项目文件**（如 `MyLib.vcxproj`） |
| **`.mwc`** | **工作区文件 (Workspace File)**：定义了一个工作区，其中列出了构成整个项目的所有 `.mpc` 项目文件，也可能包含对其他子目录或 `.mwc` 文件的引用。 | **解决方案文件**（如 `MyApp.sln`）     |
| **`.mpb`** | **基础项目文件 (Base Project File)**：作为可被多个 `.mpc` 项目文件**继承**的“模板”或“基类”，用于统一配置公共的编译选项、包含路径等，避免重复定义。 | **属性表**（如 `Common.props`）        |
| **`.mwb`** | **基础工作区文件 (Base Workspace File)**：与 `.mpb` 类似，为其他工作区（`.mwc`）提供可继承的公共配置。 | **基础解决方案配置**                   |


## 工作区文件 (.mwc) 开发

工作区文件结构非常清晰，基本骨架如下：

```shell
// 这是注释，以双斜线开头

// 声明一个工作区，可选名称，可继承基础工作区（.mwb文件）
workspace(MyProject) : some_base_workspace {
    
    // 1. 直接列出需要包含的 .mpc 项目文件
    my_library.mpc
    my_executable.mpc
    
    // 2. 列出目录路径，MPC会递归搜索目录下所有 .mpc 文件
    /path/to/other_modules
    
    // 3. 包含另一个 .mwc 文件（嵌套工作区）
    sub_workspace.mwc
    
    // 4. 使用赋值语句控制生成方式（只影响花括号内的项目）
    static {
        cmdline += -static          // 为内部项目添加命令行选项
        some_static_lib.mpc
    }
    
    // 5. 排除特定类型下的某些项目
    exclude(vc6, vc7, nmake) {     // 在 vc6/vc7/nmake 下排除这些项目
        platform_specific.mpc
    }
}
```

## 项目文件 (.mpc) 开发

项目文件稍微复杂一些，但同样遵循清晰的模式：

```shell
// 项目声明：可指定名称，可继承一个或多个基础项目（.mpb或.mpc文件）
project(my_project) : aceexe, another_base {
    
    // 可执行文件配置（三种目标类型选一种）
    exename = my_app              // 生成可执行文件
    // sharedname = my_lib        // 生成共享库（动态库）
    // staticname = my_lib        // 生成静态库
    
    // 编译配置
    includes += .                 // 头文件搜索路径
    includes += ../common
    
    libpaths += /usr/local/lib    // 库文件搜索路径
    libs += pthread               // 链接的库
    
    macros += USE_DEBUG           // 预定义宏
    
    // 源文件列表（组件列表）
    Source_Files {
        main.cpp
        helper.cpp
        src/*.cpp                 // 支持通配符
    }
    
    Header_Files {
        my_app.h
        include/*.h
    }
    
    // 如果 recurse=1，MPC会递归遍历目录下所有匹配文件[citation:2]
    recurse = 1
}
```

项目类型选择：MPC 会根据配置自动判断目标类型：

- 指定 exename → 生成可执行文件

- 指定 sharedname → 生成动态库

- 指定 staticname → 生成静态库

不指定名称 → MPC 会搜索 main 函数来决定类型

### MPC文件常用赋值关键字

| 关键字       | 作用                   | 示例                             |
| :----------- | :--------------------- | :------------------------------- |
| `exename`    | 生成的可执行文件名     | `exename = myapp`                |
| `sharedname` | 生成的共享库文件名     | `sharedname = mylib`             |
| `includes`   | 头文件搜索路径         | `includes += /usr/local/include` |
| `libpaths`   | 库文件搜索路径         | `libpaths += ../lib`             |
| `libs`       | 需要链接的库           | `libs += pthread`                |
| `macros`     | 预定义宏               | `macros += DEBUG_MODE`           |
| `recurse`    | 是否递归扫描目录       | `recurse = 1`                    |
| `requires`   | 生成项目所需特性       | `requires += ssl`                |
| `avoids`     | 生成项目需要避免的特性 | `avoids += vc6`                  |
| `after`      | 指定项目依赖顺序       | `after = other_project`          |


### MPC支持的操作符

MPC 支持三种赋值操作符：

| 操作符 | 含义             | 示例              |
| :----- | :--------------- | :---------------- |
| `=`    | 直接赋值（覆盖） | `libs = pthread`  |
| `+=`   | 追加值           | `libs += dl`      |
| `-=`   | 删除值           | `libs -= pthread` |

> 💡 **注意**：这些操作符可以混合使用，让你可以灵活地在不同继承层级中调整配置。