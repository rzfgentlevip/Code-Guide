---
title: 6、MWC 工具构建项目命令和教程总结
createTime: 2026/03/26 16:44:43
permalink: /cplus/tools-must/mpc/MWC 工具构建项目命令和教程总结/
order: 6
icon: vscode-icons:folder-type-cmake-opened
---


# MWC 工具构建项目命令和教程总结

MWC (Makefile, Project and Workspace Creator) 是 MPC 工具集中的**工作区生成器**，用于读取 `.mwc` 配置文件，自动生成各种编译环境下的项目文件（如 Visual Studio 解决方案、GNU Makefile 等）。

---

## 📋 一、MWC 工具概述

### 1.1 基本概念

| 工具    | 脚本名称             | 功能                                                     |
| ------- | -------------------- | -------------------------------------------------------- |
| **MPC** | `mpc.pl` / `mpc-ace` | 项目生成器：读取 `.mpc` 文件，生成单个项目文件           |
| **MWC** | `mwc.pl` / `mwc-ace` | 工作区生成器：读取 `.mwc` 文件，生成整个工作区及所有项目 |

**核心特性**：

- 一次编写配置文件，生成多种平台的项目文件
- 支持项目继承和公共配置复用
- 支持条件编译和特性控制
- 自动处理跨平台差异（路径分隔符、库命名等）

### 1.2 工作流程

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  .mwc 文件      │     │  .mpc 文件      │     │  生成的项目文件   │
│  (工作区配置)    │ ──▶ │  (项目配置)      │ ──▶ │  .sln/.vcxproj  │
└─────────────────┘     └─────────────────┘     │  Makefile       │
                                                  │  .dsp/.dsw      │
                                                  └─────────────────┘
```

---

## 🚀 二、MWC 命令基本用法

### 2.1 命令格式

```bash
mwc-ace [选项] [输入文件.mwc ...]
```

如果不指定输入文件，MWC 会**递归搜索**当前目录及子目录下所有 `.mpc` 文件，自动组合成一个工作区进行处理 。

### 2.2 常用命令示例

```bash
# 基本用法：生成默认类型的项目
mwc-ace

# 指定项目类型（Visual Studio 2010）
mwc-ace -type vc10

# 指定工作区文件
mwc-ace my_workspace.mwc

# 同时生成多种项目类型
mwc-ace -type vc10 -type gnuace

# 递归处理所有子目录
mwc-ace -recurse

# 生成静态库版本的项目
mwc-ace -static -type vc10

# 指定输出目录
mwc-ace -into ./build
```

---

## ⚙️ 三、MWC 核心选项详解

### 3.1 项目类型选项 (-type)

指定生成的项目文件类型，可多次使用以同时生成多种类型 ：

| 类型参数   | 说明                   | 生成文件          |
| ---------- | ---------------------- | ----------------- |
| `vc6`      | Visual C++ 6.0         | `.dsp`/`.dsw`     |
| `vc7`      | Visual C++ 7.0 (2002)  | `.vcproj`/`.sln`  |
| `vc8`      | Visual C++ 8.0 (2005)  | `.vcproj`/`.sln`  |
| `vc9`      | Visual C++ 9.0 (2008)  | `.vcproj`/`.sln`  |
| `vc10`     | Visual C++ 10.0 (2010) | `.vcxproj`/`.sln` |
| `gnuace`   | GNU Make (ACE扩展)     | `Makefile`        |
| `nmake`    | Microsoft NMake        | `Makefile`        |
| `automake` | GNU Automake           | `Makefile.am`     |
| `borland`  | Borland C++ Builder    | `.bpr`            |

```bash
# 同时生成 VS2010 和 Makefile
mwc-ace -type vc10 -type gnuace workspace.mwc
```

### 3.2 路径和搜索选项

| 选项                 | 说明                 | 示例                                    |
| -------------------- | -------------------- | --------------------------------------- |
| `-include dir`       | 添加基础项目搜索路径 | `-include mpc -include $QUICKFAST_ROOT` |
| `-recurse`           | 递归搜索所有输入文件 | `-recurse`                              |
| `-exclude dirs`      | 排除指定目录或文件   | `-exclude tests,examples`               |
| `-relative name=var` | 相对路径替换规则     | `-relative PROJ_TOP=/path`              |

### 3.3 特性控制选项

| 选项                 | 说明                         | 示例                               |
| -------------------- | ---------------------------- | ---------------------------------- |
| `-feature_file file` | 指定特性配置文件             | `-feature_file liquibook.features` |
| `-features list`     | 设置特性开关                 | `-features "ssl=1,qos=0"`          |
| `-base project`      | 强制所有项目继承指定基础项目 | `-base common_base`                |
| `-static`            | 生成静态库版本               | `-static`                          |

**特性文件示例** (`liquibook.features`) ：

```mpc
feature debug
    description = "Enable debug logging"
    default = 0

feature ssl
    description = "Enable SSL/TLS support"
    default = 0
```

### 3.4 输出控制选项

| 选项                     | 说明                                      | 示例                        |
| ------------------------ | ----------------------------------------- | --------------------------- |
| `-into dir`              | 输出到镜像目录结构                        | `-into /output/path`        |
| `-name_modifier pattern` | 修改生成的文件名                          | `-name_modifier '*_Static'` |
| `-apply_project`         | 同时修改项目名称（配合 `-name_modifier`） | `-apply_project`            |
| `-notoplevel`            | 不生成顶层工作区文件                      | `-notoplevel`               |
| `-nocomments`            | 不在生成文件中添加注释                    | `-nocomments`               |

### 3.5 变量处理选项

| 选项                       | 说明             | 示例                                             |
| -------------------------- | ---------------- | ------------------------------------------------ |
| `-expand_vars`             | 直接展开所有变量 | `-expand_vars`                                   |
| `-use_env`                 | 使用环境变量替换 | `-use_env`                                       |
| `-value_template name=val` | 修改模板输入值   | `-value_template "configurations=Release Debug"` |
| `-value_project name=val`  | 修改项目变量赋值 | `-value_project "libs+=ssl"`                     |

### 3.6 高级选项

| 选项           | 说明                                   | 示例                        |
| -------------- | -------------------------------------- | --------------------------- |
| `-hierarchy`   | 生成层级工作区结构（make类型默认启用） | `-hierarchy`                |
| `-gendot`      | 生成 Graphviz `.dot` 依赖图文件        | `-gendot`                   |
| `-workers N`   | 指定并行处理进程数                     | `-workers 4`                |
| `-version`     | 显示 MPC 版本                          | `-version`                  |
| `-global file` | 指定全局输入文件                       | `-global config/global.mpb` |

---

## 📁 四、输入文件类型详解

### 4.1 项目文件 (.mpc)

定义单个编译目标的配置 ：

```mpc
project(myapp) : common_base {
    exename = myapp                    # 生成可执行文件
    includes += include                # 头文件路径
    libs += pthread                    # 链接库
    macros += DEBUG_MODE               # 预定义宏
    
    Source_Files {
        src/*.cpp                      # 源文件
    }
    
    Header_Files {
        include/*.h                    # 头文件
    }
}
```

### 4.2 工作区文件 (.mwc)

定义项目集合和工作区结构 ：

```mpc
workspace(MyProject) : base_workspace {
    # 全局命令行选项
    cmdline += -include mpc
    cmdline += -expand_vars
    
    # 列出项目文件
    mylib.mpc
    myapp.mpc
    
    # 列出目录（递归搜索）
    modules/
    
    # 嵌套其他工作区
    sub_workspace.mwc
    
    # 排除特定类型下的项目
    exclude(vc6, nmake) {
        windows_specific.mpc
    }
    
    # 作用域控制
    static_libs {
        cmdline += -static
        static_lib.mpc
    }
}
```

### 4.3 基础项目文件 (.mpb)

公共配置模板，供多个项目继承 ：

```mpc
project(common_base) {
    includes += /usr/local/include
    libs += pthread
    cflags += -Wall -O2
}
```

---

## 🛠️ 五、实战教程

### 5.1 基础教程：生成 Visual Studio 项目

**步骤 1：创建项目目录结构**

```
myproject/
├── mpc/
│   └── common.mpb           # 基础配置
├── src/
│   ├── main.cpp
│   └── utils.cpp
├── include/
│   └── utils.h
├── mylib.mpc                # 库项目
├── myapp.mpc                # 可执行项目
└── myproject.mwc            # 工作区
```

**步骤 2：编写基础项目文件** (`mpc/common.mpb`)

```mpc
project(common) {
    includes += ../include
    cflags += /W4 /EHsc
    macros += _CRT_SECURE_NO_WARNINGS
}
```

**步骤 3：编写库项目** (`mylib.mpc`)

```mpc
project(mylib) : common {
    staticname = mylib
    Source_Files {
        ../src/utils.cpp
    }
    Header_Files {
        ../include/utils.h
    }
}
```

**步骤 4：编写可执行项目** (`myapp.mpc`)

```mpc
project(myapp) : common {
    exename = myapp
    libs += mylib
    after += mylib
    
    Source_Files {
        ../src/main.cpp
    }
}
```

**步骤 5：编写工作区文件** (`myproject.mwc`)

```mpc
workspace(MyProject) {
    cmdline += -include mpc
    cmdline += -expand_vars
    
    mylib.mpc
    myapp.mpc
}
```

**步骤 6：生成 Visual Studio 项目**

```bash
mwc-ace -type vc10 myproject.mwc
```

执行后生成：

- `mylib.vcxproj`
- `myapp.vcxproj`
- `MyProject.sln`

### 5.2 中级教程：条件编译和特性控制

**特性文件** (`config/liquibook.features`)

```mpc
feature debug
    default = 0
    
feature ssl
    default = 0
    requires = openssl
```

**项目文件使用特性** (`ssl_app.mpc`)

```mpc
project(ssl_app) : common {
    requires += ssl               # 只在启用 SSL 时生成
    exename = ssl_app
    libs += ssl
    Source_Files { ssl_main.cpp }
}
```

**生成时启用特性**

```bash
mwc-ace -type vc10 -features "ssl=1" myproject.mwc
```

### 5.3 高级教程：静态库和动态库同时生成

由于 MPC 一次只能生成一种库类型（动态或静态），需要运行两次 ：

```bash
# 生成动态库版本
mwc-ace -type vc10 myproject.mwc

# 生成静态库版本（文件名添加 _Static 后缀）
mwc-ace -type vc10 -static -name_modifier '*_Static' -apply_project myproject.mwc
```

**在工作区内使用作用域控制部分静态项目** ：

```mpc
workspace {
    # 动态库项目（默认）
    mylib_dynamic.mpc
    myapp.mpc
    
    # 静态库项目（仅这部分使用 -static）
    static_libs {
        cmdline += -static
        mylib_static.mpc
    }
}
```

### 5.4 跨平台教程：同时生成多种项目类型

```bash
# Windows: Visual Studio 2010
mwc-ace -type vc10 myproject.mwc

# Windows: NMake Makefile
mwc-ace -type nmake myproject.mwc

# Linux: GNU Makefile
mwc-ace -type gnuace myproject.mwc

# macOS: GNU Makefile（同样使用 gnuace）
mwc-ace -type gnuace myproject.mwc
```

---

## 🔧 六、常见问题解决

### 6.1 Perl 关联问题（Windows）

**现象**：执行 `mwc.pl` 时提示没有参数，无法识别命令行选项 。

**原因**：Perl 文件关联配置不正确，缺少 `%*` 参数传递。

**解决方法**：检查并修改注册表：

1. `HKEY_CLASSES_ROOT\.pl` 的默认值应为 `Perl`

2. `HKEY_CLASSES_ROOT\Perl\Shell\Open\Command` 的默认值应为：

   ```
   "C:\Perl\bin\perl.exe" "%1" %*
   ```

3. 值类型必须为 `REG_EXPAND_SZ`

### 6.2 找不到基础项目文件

**现象**：生成时提示找不到基础项目。

**解决方法**：

```bash
# 使用 -include 添加搜索路径
mwc-ace -include mpc -include $QUICKFAST_ROOT myproject.mwc
```

### 6.3 静态库依赖问题

**现象**：生成静态库时，库之间没有生成依赖关系 。

**解决方法**：设置环境变量

```bash
export MPC_DEPENDENCY_COMBINED_STATIC_LIBRARY=1
mwc-ace -static -type vc10 myproject.mwc
```

### 6.4 输出目录配置

**问题**：Debug/Release 版本输出到同一目录 。

**解决方法**：在项目文件中配置：

```mpc
project {
    specific(nmake, vc6, vc7, vc71, vc8) {
        Debug::install = Debug
        Release::install = Release
    }
}
```

---

## 📊 七、MWC 命令速查表

| 任务               | 命令                                               |
| ------------------ | -------------------------------------------------- |
| 生成 VS2010 项目   | `mwc-ace -type vc10`                               |
| 生成 GNU Makefile  | `mwc-ace -type gnuace`                             |
| 同时生成多种类型   | `mwc-ace -type vc10 -type gnuace`                  |
| 生成静态库版本     | `mwc-ace -static -type vc10`                       |
| 添加特性开关       | `mwc-ace -features "ssl=1"`                        |
| 指定工作区文件     | `mwc-ace my_workspace.mwc`                         |
| 递归搜索所有子目录 | `mwc-ace -recurse`                                 |
| 排除特定目录       | `mwc-ace -exclude tests,examples`                  |
| 修改文件名后缀     | `mwc-ace -name_modifier '*_Static' -apply_project` |
| 输出到指定目录     | `mwc-ace -into ./build`                            |
| 查看版本           | `mwc-ace -version`                                 |

---

## 📚 八、学习资源

- **官方文档**：`$ACE_ROOT/bin/MakeProjectCreator/` 下的 `README` 和 `USAGE`
- **命令行帮助**：`mwc-ace -help`
- **示例项目**：ACE/TAO 框架自带的 `.mwc` 和 `.mpc` 文件
- **Debian manpage**：`man mwc-ace` 

> 💡 **核心要点**：MWC 的核心价值在于"一次编写，多平台生成"。通过合理组织 `.mpc` 和 `.mwc` 文件，结合特性控制和条件编译，可以极大简化跨平台 C++ 项目的构建管理。