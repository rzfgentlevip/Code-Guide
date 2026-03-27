---
title: 4、mpc配置文件指令
createTime: 2026/03/26 10:39:52
permalink: /cplus/tools-must/mpc/mpc配置文件指令/
order: 4
icon: material-icon-theme:folder-config
---


# MPC 配置文件指令完整总结

MPC (Make Project Creator) 工具集包含多种配置文件和指令，下面按照文件类型和指令类别进行系统总结。

---

## 📁 一、配置文件类型概览

| 文件类型           | 后缀        | 作用             | 主要指令                                |
| ------------------ | ----------- | ---------------- | --------------------------------------- |
| **项目文件**       | `.mpc`      | 定义单个编译目标 | `project`、`exename`、`Source_Files` 等 |
| **工作区文件**     | `.mwc`      | 组织多个项目     | `workspace`、`exclude`、`cmdline` 等    |
| **基础项目文件**   | `.mpb`      | 公共配置模板     | `project`（作为基类）                   |
| **基础工作区文件** | `.mwb`      | 工作区公共配置   | `workspace`（作为基类）                 |
| **特性文件**       | `.features` | 定义编译特性     | `feature`、`default`                    |

---

## 🔧 二、项目文件 (.mpc) 指令

### 2.1 核心声明指令

| 指令           | 语法                            | 说明                         | 示例                              |
| -------------- | ------------------------------- | ---------------------------- | --------------------------------- |
| **project**    | `project(name) : bases { ... }` | 声明一个项目，可继承基础项目 | `project(myapp) : common { ... }` |
| **exename**    | `exename = name`                | 生成可执行文件               | `exename = myapp`                 |
| **sharedname** | `sharedname = name`             | 生成共享库（动态库）         | `sharedname = mylib`              |
| **staticname** | `staticname = name`             | 生成静态库                   | `staticname = mylib`              |
| **libout**     | `libout = path`                 | 库文件输出路径               | `libout = ../lib`                 |
| **dllout**     | `dllout = path`                 | 动态库输出路径               | `dllout = ../bin`                 |

### 2.2 文件列表组件

| 组件               | 用途                              | 示例                                  |
| ------------------ | --------------------------------- | ------------------------------------- |
| **Source_Files**   | 源文件列表（`.cpp`、`.c`、`.cc`） | `Source_Files { main.cpp src/*.cpp }` |
| **Header_Files**   | 头文件列表（`.h`、`.hpp`）        | `Header_Files { include/*.h }`        |
| **Resource_Files** | 资源文件列表                      | `Resource_Files { app.rc }`           |
| **IDL_Files**      | IDL 文件列表（CORBA）             | `IDL_Files { interface.idl }`         |

### 2.3 编译配置指令

| 指令          | 说明                              | 示例                         |
| ------------- | --------------------------------- | ---------------------------- |
| **includes**  | 头文件搜索路径                    | `includes += ../include`     |
| **libpaths**  | 库文件搜索路径                    | `libpaths += /usr/local/lib` |
| **libs**      | 需要链接的库（自动添加前缀/后缀） | `libs += pthread`            |
| **lit_libs**  | 需要链接的库（不添加修饰符）      | `lit_libs += mylib`          |
| **pure_libs** | 需要链接的库（无前缀/后缀）       | `pure_libs += m`             |
| **macros**    | 预定义宏                          | `macros += DEBUG_MODE`       |
| **cflags**    | C 编译器选项                      | `cflags += -Wall -O2`        |
| **cxxflags**  | C++ 编译器选项                    | `cxxflags += -std=c++11`     |
| **ldflags**   | 链接器选项                        | `ldflags += -Wl,-rpath,.`    |

### 2.4 控制指令

| 指令           | 值       | 说明               | 示例                                |
| -------------- | -------- | ------------------ | ----------------------------------- |
| **recurse**    | 0/1      | 是否递归扫描目录   | `recurse = 1`                       |
| **requires**   | 特性列表 | 需要满足的特性     | `requires += ssl`                   |
| **avoids**     | 特性列表 | 需要避免的特性     | `avoids += vc6`                     |
| **after**      | 项目列表 | 指定项目依赖顺序   | `after = other_project`             |
| **version**    | 版本号   | 库或可执行文件版本 | `version = 1.2.0`                   |
| **install**    | 0/1      | 是否安装可执行文件 | `install = 1`                       |
| **pch_header** | 文件名   | 预编译头文件       | `pch_header = stdafx.h`             |
| **pch_source** | 文件名   | 预编译头源文件     | `pch_source = stdafx.cpp`           |
| **postbuild**  | 命令     | 构建后执行命令     | `postbuild = copy $(TARGET) ../bin` |

### 2.5 赋值操作符

| 操作符 | 含义             | 示例              |
| ------ | ---------------- | ----------------- |
| `=`    | 直接赋值（覆盖） | `libs = pthread`  |
| `+=`   | 追加值           | `libs += dl`      |
| `-=`   | 删除值           | `libs -= pthread` |

---

## 📂 三、工作区文件 (.mwc) 指令

### 3.1 核心声明指令

| 指令          | 语法                             | 说明               | 示例                                |
| ------------- | -------------------------------- | ------------------ | ----------------------------------- |
| **workspace** | `workspace(name) : base { ... }` | 声明工作区         | `workspace(MyProject) { ... }`      |
| **exclude**   | `exclude(types) { ... }`         | 排除特定类型的项目 | `exclude(vc6, nmake) { test.mpc }`  |
| **implicit**  | `implicit = value`               | 启用隐式项目生成   | `implicit = 1` 或 `implicit = base` |
| **cmdline**   | `cmdline += options`             | 传递命令行选项     | `cmdline += -include mpc`           |
| **specific**  | `specific(type) { ... }`         | 特定类型的配置     | `specific(make) { ... }`            |

### 3.2 作用域控制

| 结构                 | 说明                 | 示例                                |
| -------------------- | -------------------- | ----------------------------------- |
| `{ }`                | 花括号定义作用域     | `static { cmdline += -static }`     |
| `specific(type) { }` | 仅对特定项目类型生效 | `specific(vc10) { ... }`            |
| 自定义作用域         | 任意命名的配置块     | `debug_build { cmdline += -debug }` |

### 3.3 文件/目录包含

| 方式            | 说明                      | 示例                |
| --------------- | ------------------------- | ------------------- |
| 直接列出 `.mpc` | 包含单个项目文件          | `my_library.mpc`    |
| 列出目录路径    | 递归搜索目录下所有 `.mpc` | `src/`              |
| 包含 `.mwc`     | 嵌套其他工作区            | `sub_workspace.mwc` |

---

## 🏗️ 四、基础文件指令 (.mpb / .mwb)

基础文件与普通项目/工作区文件使用相同的指令，但通常只包含公共配置：

```mpc
// 基础项目文件示例 (common.mpb)
project(common) {
    includes += ../include
    includes += /usr/local/include
    libs += pthread
    macros += _REENTRANT
    cflags += -Wall -O2
}

// 基础工作区文件示例 (base.mwb)
workspace(base) {
    cmdline += -include mpc
    cmdline += -expand_vars
    cmdline += -use_env
}
```

---

## ⚙️ 五、特性文件 (.features) 指令

| 指令            | 语法                 | 说明         | 示例                                 |
| --------------- | -------------------- | ------------ | ------------------------------------ |
| **feature**     | `feature name`       | 声明一个特性 | `feature ssl`                        |
| **default**     | `default = value`    | 设置默认值   | `default = 0`                        |
| **description** | `description = text` | 特性描述     | `description = "Enable SSL support"` |
| **requires**    | `requires = feature` | 依赖其他特性 | `requires = openssl`                 |

```mpc
// liquibook.features 示例
feature debug
    description = "Enable debug logging"
    default = 0

feature ssl
    description = "Enable SSL/TLS support"
    default = 0
    requires = openssl

feature profiling
    description = "Enable performance profiling"
    default = 0
```

---

## 🛠️ 六、常用 cmdline 选项（用于 .mwc 或命令行）

### 6.1 路径和文件选项

| 选项                 | 说明                   | 示例                               |
| -------------------- | ---------------------- | ---------------------------------- |
| `-include dir`       | 添加基础项目搜索路径   | `-include mpc`                     |
| `-feature_file file` | 指定特性配置文件       | `-feature_file liquibook.features` |
| `-global file`       | 指定全局输入文件       | `-global config/global.mpb`        |
| `-template file`     | 指定模板文件           | `-template mytemplate`             |
| `-ti type:file`      | 指定特定类型的模板输入 | `-ti dll_exe:vc8exe`               |

### 6.2 特性控制选项

| 选项             | 说明                         | 示例                      |
| ---------------- | ---------------------------- | ------------------------- |
| `-features list` | 设置特性开关                 | `-features "ssl=1,qos=0"` |
| `-base project`  | 强制所有项目继承指定基础项目 | `-base common_base`       |
| `-static`        | 生成静态库版本               | `-static`                 |

### 6.3 变量处理选项

| 选项                 | 说明             | 示例                       |
| -------------------- | ---------------- | -------------------------- |
| `-expand_vars`       | 直接展开所有变量 | `-expand_vars`             |
| `-use_env`           | 使用环境变量替换 | `-use_env`                 |
| `-relative name=var` | 相对路径替换规则 | `-relative PROJ_TOP=/path` |

### 6.4 输出控制选项

| 选项                       | 说明                             | 示例                                             |
| -------------------------- | -------------------------------- | ------------------------------------------------ |
| `-type types`              | 指定生成的项目类型               | `-type vc10 -type gnuace`                        |
| `-into dir`                | 指定输出目录                     | `-into /output/path`                             |
| `-name_modifier pattern`   | 修改生成的文件名                 | `-name_modifier '*_Static'`                      |
| `-apply_project`           | 配合 `-name_modifier` 修改项目名 | `-apply_project`                                 |
| `-value_template name=val` | 修改模板输入值                   | `-value_template "configurations=Release Debug"` |
| `-value_project name=val`  | 修改项目变量赋值                 | `-value_project "libs+=ssl"`                     |

### 6.5 行为控制选项

| 选项                | 说明                            | 示例                      |
| ------------------- | ------------------------------- | ------------------------- |
| `-recurse`          | 递归搜索所有输入文件            | `-recurse`                |
| `-exclude dirs`     | 排除目录                        | `-exclude tests,examples` |
| `-notoplevel`       | 不生成顶层工作区文件            | `-notoplevel`             |
| `-noreldefs`        | 不生成默认相对路径定义          | `-noreldefs`              |
| `-genins`           | 生成 `.ins` 安装文件            | `-genins`                 |
| `-make_coexistence` | 多个 make 类型共存时区分命名    | `-make_coexistence`       |
| `-hierarchy`        | 生成层级工作区结构              | `-hierarchy`              |
| `-gendot`           | 生成 Graphviz `.dot` 依赖图文件 | `-gendot`                 |
| `-language lang`    | 指定编程语言                    | `-language cplusplus`     |

---

## 📊 七、支持的项目类型

| 类型       | 说明                   | 生成文件          |
| ---------- | ---------------------- | ----------------- |
| `vc6`      | Visual C++ 6.0         | `.dsp`/`.dsw`     |
| `vc7`      | Visual C++ 7.0 (2002)  | `.vcproj`/`.sln`  |
| `vc71`     | Visual C++ 7.1 (2003)  | `.vcproj`/`.sln`  |
| `vc8`      | Visual C++ 8.0 (2005)  | `.vcproj`/`.sln`  |
| `vc9`      | Visual C++ 9.0 (2008)  | `.vcproj`/`.sln`  |
| `vc10`     | Visual C++ 10.0 (2010) | `.vcxproj`/`.sln` |
| `vc11`     | Visual C++ 11.0 (2012) | `.vcxproj`/`.sln` |
| `vc12`     | Visual C++ 12.0 (2013) | `.vcxproj`/`.sln` |
| `gnuace`   | GNU Make (ACE扩展)     | `Makefile`        |
| `nmake`    | Microsoft NMake        | `Makefile`        |
| `bmake`    | Borland Make           | `Makefile`        |
| `automake` | GNU Automake           | `Makefile.am`     |
| `borland`  | Borland C++ Builder    | `.bpr`            |
| `em3`      | eMbedded Visual C++    | 嵌入式平台        |

---

## 💡 八、指令使用速查表

### 文件类型判断

| 目标类型   | 使用的指令   | 必须项         |
| ---------- | ------------ | -------------- |
| 可执行文件 | `exename`    | 至少一个源文件 |
| 动态库     | `sharedname` | 至少一个源文件 |
| 静态库     | `staticname` | 至少一个源文件 |

### 常见组合模式

```mpc
// 1. 最小可执行项目
project {
    exename = myapp
    Source_Files { main.cpp }
}

// 2. 带配置的可执行项目
project(myapp) : common {
    exename = myapp
    includes += include
    libs += pthread
    Source_Files { src/*.cpp }
}

// 3. 库项目
project(mylib) {
    staticname = mylib
    sharedname = mylib
    includes += include
    Source_Files { src/*.cpp }
    version = 1.0.0
}

// 4. 工作区
workspace(MyProject) {
    cmdline += -include mpc
    mylib.mpc
    myapp.mpc
    specific(vc10) {
        cmdline += -value_template "configurations=Debug;Release"
    }
}
```

---

## 📚 参考资源

- **官方文档**：`$ACE_ROOT/bin/MakeProjectCreator/` 目录下的 `README` 和 `USAGE`
- **命令行帮助**：`mpc.pl -help`、`mwc.pl -help`
- **示例文件**：ACE/TAO 项目中的 `.mpc` 和 `.mwc` 文件

> 💡 **提示**：MPC 指令大小写不敏感（如 `Source_Files` 和 `source_files` 等效），但习惯上使用首字母大写的命名风格。

