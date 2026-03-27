---
title: 3、wmc配置文件指令
createTime: 2026/03/26 10:28:14
permalink: /cplus/tools-must/mpc/wmc配置文件指令/
order: 3
icon: material-icon-theme:astro-config
---

## 📋 一、核心结构指令

### 1. `workspace(name) : base_workspace`

声明一个工作区，并可选地指定名称和继承的基础工作区。

```shell
// 无名称的工作区（使用文件名作为名称）
workspace {
    ...
}

// 有名称的工作区
workspace(MyProject) {
    ...
}

// 继承基础工作区（.mwb 文件）
workspace(MyProject) : common_base {
    ...
}
```

| 参数             | 说明                                             |
| ---------------- | ------------------------------------------------ |
| `name`           | 可选，工作区名称，默认为 `.mwc` 文件名           |
| `base_workspace` | 可选，要继承的基础工作区名称（对应 `.mwb` 文件） |


### 2. `exclude(types) { ... }`

排除特定的项目类型，使其不生成项目文件。

```shell
// 排除所有类型
exclude {
    platform_specific.mpc
}

// 排除特定类型（如 VC6、VC7、NMake）
exclude(vc6, vc7, nmake) {
    windows_only.mpc
}
```

**说明**：在 `exclude` 关键字后的括号中指定需要排除的项目类型列表，花括号内列出要排除的 `.mpc` 文件或目录路径。如果不指定任何类型，表示排除所有项目类型。


### 3. `implicit = value`

启用隐式项目生成，为没有 `.mpc` 文件的目录自动生成默认项目。

```shell
workspace {
    implicit = 1              // 启用隐式项目生成
    // 或
    implicit = example_base   // 使用指定的基础项目
    
    src/
    test/
}
```

**说明**：

- 设置为 `1`：为每个目录自动生成默认项目
- 设置为基础项目名称：隐式生成的项目继承该基础项目
- 如果目录下没有可用的源文件，则不会生成项目


## ⚙️ 二、命令行选项传递指令

### `cmdline += options`

在 `.mwc` 文件内部传递命令行选项给 MPC 工具。

```shell
workspace {
    // 添加基础项目搜索路径
    cmdline += -include mpc
    cmdline += -include $QUICKFAST_ROOT
    
    // 指定特性配置文件
    cmdline += -feature_file liquibook.features
    
    // 展开变量
    cmdline += -expand_vars
    
    // 使用环境变量
    cmdline += -use_env
    
    // 指定项目类型
    cmdline += -type vc10
    
    // 修改模板值
    cmdline += -value_template "configurations=Release Debug"
}
```

**注意事项**：

- 以下选项在 `cmdline` 中会被忽略：`-type`、`-recurse`、`-noreldefs`、`-make_coexistence`、`-genins`、`-into`、`-language`
- 环境变量可以通过 `$NAME` 方式访问


## 🎯 三、作用域控制指令

### `specific(type) { ... }`

为特定项目类型指定配置，仅对指定类型生效。

```shell
workspace {
    // 全局配置
    cmdline += -include common
    
    // 仅对 make 类型生效的配置
    specific(make) {
        cmdline += -value_template "configurations=Release Debug"
    }
    
    // 仅对 VC10 类型生效的配置
    specific(vc10) {
        cmdline += -value_template "configurations=Debug Release"
    }
}
```


### 自定义作用域（如 `static { }`）

可以在工作区内创建任意名称的作用域，用于限定配置的影响范围。

```shell
workspace {
    // 所有项目都会受到全局 cmdline 的影响
    cmdline += -include common
    
    // 只有在这个作用域内的项目会添加 -static 选项
    static_libs {
        cmdline += -static
        my_static_lib.mpc
        another_static_lib.mpc
    }
    
    // 这个项目不受 -static 选项影响
    my_executable.mpc
}
```


## 📁 四、文件/目录包含指令

### 直接列出 `.mpc` 文件

```shell
workspace {
    my_library.mpc
    my_executable.mpc
    ../other_project/other.mpc
}
```

### 直接列出目录

列出目录路径时，MPC 会递归搜索该目录下所有 `.mpc` 文件。

```shell
workspace {
    src/                    // 搜索 src 目录及其子目录
    ../modules/             // 搜索上级目录的 modules 目录
}
```

### 包含其他 `.mwc` 文件

```shell
workspace {
    sub_workspace.mwc       // 聚合另一个工作区
    ../common/common.mwc
}
```


## 🔧 五、常用 `cmdline` 选项速查

以下选项可以在 `.mwc` 文件中通过 `cmdline +=` 使用：

| 选项                       | 说明                             | 示例                                             |
| -------------------------- | -------------------------------- | ------------------------------------------------ |
| `-include dir`             | 添加基础项目搜索路径             | `-include mpc`                                   |
| `-feature_file file`       | 指定特性配置文件                 | `-feature_file liquibook.features`               |
| `-features list`           | 设置特性开关                     | `-features "ssl=1,qos=0"`                        |
| `-expand_vars`             | 直接展开所有变量                 | `-expand_vars`                                   |
| `-use_env`                 | 使用环境变量替换                 | `-use_env`                                       |
| `-base project`            | 强制所有项目继承指定基础项目     | `-base common_base`                              |
| `-static`                  | 生成静态库版本                   | `-static`                                        |
| `-value_template name=val` | 修改模板输入值                   | `-value_template "configurations=Release Debug"` |
| `-value_project name=val`  | 修改项目变量赋值                 | `-value_project "libs+=ssl"`                     |
| `-name_modifier pattern`   | 修改生成的文件名                 | `-name_modifier '*_Static'`                      |
| `-apply_project`           | 配合 `-name_modifier` 修改项目名 | `-apply_project`                                 |
| `-exclude dirs`            | 排除目录（仅对 mwc.pl 有效）     | `-exclude tests,examples`                        |
| `-recurse`                 | 递归搜索所有输入文件             | `-recurse`                                       |
| `-notoplevel`              | 不生成顶层工作区文件             | `-notoplevel`                                    |
| `-noreldefs`               | 不生成默认相对路径定义           | `-noreldefs`                                     |
| `-genins`                  | 生成 `.ins` 安装文件             | `-genins`                                        |
| `-make_coexistence`        | 多个 make 类型共存时区分命名     | `-make_coexistence`                              |
| `-hierarchy`               | 生成层级工作区结构               | `-hierarchy`                                     |
| `-into dir`                | 输出到镜像目录结构               | `-into /output/path`                             |
| `-language lang`           | 指定编程语言                     | `-language cplusplus`                            |
| `-global file`             | 指定全局输入文件                 | `-global config/global.mpb`                      |
| `-template file`           | 指定模板文件                     | `-template mytemplate`                           |
| `-ti type:file`            | 指定特定类型的模板输入文件       | `-ti dll_exe:vc8exe`                             |
| `-relative name=var`       | 相对路径替换规则                 | `-relative PROJ_TOP=/path`                       |
| `-gendot`                  | 生成 Graphviz `.dot` 依赖图文件  | `-gendot`                                        |


## 💡 六、完整示例

```shell
// 声明工作区，继承基础工作区
workspace(liquibook) : ace_workspace {
  
  // ========== 全局命令行选项 ==========
  // 添加基础项目搜索路径
  cmdline += -include mpc 
  cmdline += -include $QUICKFAST_ROOT
  
  // 指定特性配置文件
  cmdline += -feature_file liquibook.features
  
  // 展开变量并使用环境变量
  cmdline += -expand_vars
  cmdline += -use_env
  
  // 指定生成的项目类型
  cmdline += -type vc10
  cmdline += -type gnuace
  
  // ========== 特定平台配置 ==========
  specific(make) {
    cmdline += -value_template "configurations=Release Debug"
  }
  
  specific(vc10) {
    cmdline += -value_template "configurations=Debug;Release"
  }
  
  // ========== 作用域控制 ==========
  // 静态库项目作用域
  static_libs {
    cmdline += -static
    libs/lib1.mpc
    libs/lib2.mpc
  }
  
  // ========== 排除项 ==========
  // 在 VC6 环境下排除这些项目
  exclude(vc6) {
    modern_features.mpc
  }
  
  // ========== 包含项目 ==========
  // 直接列出项目文件
  apps/main_app.mpc
  apps/tool.mpc
  
  // 包含目录（递归搜索）
  tests/
  
  // 包含其他工作区
  submodules/helper.mwc
}
```


## 📚 参考资源

- MPC 官方文档：`$ACE_ROOT/bin/MakeProjectCreator/USAGE`
- 命令行帮助：`mwc.pl -help`