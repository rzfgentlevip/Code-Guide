---
title: 8、common.mpb 公共配置内容指南
createTime: 2026/03/26 16:46:51
permalink: /cplus/tools-must/mpc/common.mpb 公共配置内容指南/
order: 8
icon: vscode-icons:file-type-snakemake
---


# common.mpb 公共配置内容指南

`common.mpb` 是 MPC 中的**基础项目文件**，用于存放多个项目共享的公共配置。它就像一个"模板"，让其他项目通过继承获得这些配置。

---

## 📋 一、common.mpb 常见内容

### 1. 基本结构

```mpc
project(common) {
    # 配置内容
}
```

---

## 🔧 二、常用配置项

### 2.1 编译器选项

```mpc
project(common) {
    # C++ 编译器选项
    cxxflags += -std=c++11
    cxxflags += -Wall
    cxxflags += -Wextra
    cxxflags += -Werror
    cxxflags += -O2
    cxxflags += -g
    
    # 或者分配置设置
    Release::cxxflags += -O2 -DNDEBUG
    Debug::cxxflags += -g -O0 -DDEBUG
}
```

### 2.2 头文件搜索路径

```mpc
project(common) {
    # 公共头文件目录
    includes += include
    includes += ../common
    includes += /usr/local/include
    
    # 不同平台
    specific(prop:microsoft) {
        includes += C:/boost/include
    }
    specific(prop:gnu) {
        includes += /usr/include/boost
    }
}
```

### 2.3 预定义宏

```mpc
project(common) {
    # 版本信息
    macros += VERSION=1.0.0
    macros += PROJECT_NAME=\"MyApp\"
    
    # 平台宏
    macros += _REENTRANT
    macros += _GNU_SOURCE
    
    # 调试宏
    macros += DEBUG
    macros += _DEBUG
    
    # 特性开关
    macros += USE_STD_CXX11
}
```

### 2.4 库路径和链接库

```mpc
project(common) {
    # 公共库路径
    libpaths += /usr/local/lib
    libpaths += ../lib
    
    # 公共链接库
    libs += pthread
    libs += dl
    libs += rt
    
    # 条件链接
    requires += ssl
    libs += ssl crypto
}
```

### 2.5 版本信息

```mpc
project(common) {
    # 项目版本
    version = 1.0.0
    
    # 库版本
    major = 1
    minor = 0
    micro = 0
}
```

---

## 📝 三、完整的 common.mpb 示例

### 示例 1：基础配置

```mpc
project(common) {
    # 编译器选项
    cxxflags += -std=c++11
    cxxflags += -Wall
    cxxflags += -Wextra
    cxxflags += -fPIC
    
    # 头文件路径
    includes += include
    
    # 公共宏
    macros += _REENTRANT
    macros += _GNU_SOURCE
    
    # 公共库
    libs += pthread
    
    # 版本
    version = 1.0.0
    
    # 不同配置
    Release::cxxflags += -O2 -DNDEBUG
    Debug::cxxflags += -g -O0 -DDEBUG
}
```

### 示例 2：带平台适配的配置

```mpc
project(common) {
    # 通用配置
    cxxflags += -std=c++11 -Wall -Wextra
    includes += include
    macros += _REENTRANT
    version = 1.0.0
    
    # Windows 特定
    specific(prop:microsoft) {
        cxxflags += /EHsc /MP
        macros += WIN32
        macros += _WINDOWS
        libs += ws2_32
    }
    
    # Linux/Unix 特定
    specific(prop:gnu) {
        cxxflags += -fPIC -pthread
        macros += LINUX
        libs += pthread dl rt
    }
    
    # macOS 特定
    specific(prop:apple) {
        cxxflags += -fPIC
        macros += MACOSX
        libs += pthread
    }
    
    # 调试/发布
    Debug::cxxflags += -g -O0 -DDEBUG
    Release::cxxflags += -O2 -DNDEBUG
}
```

### 示例 3：带特性控制的配置

```mpc
project(common) {
    # 基础配置
    cxxflags += -std=c++11 -Wall
    includes += include
    
    # 根据特性添加宏
    requires += debug
    macros += DEBUG_MODE
    
    requires += ssl
    macros += HAVE_SSL
    libs += ssl crypto
    
    requires += boost
    macros += HAVE_BOOST
    includes += $(BOOST_ROOT)/include
    libpaths += $(BOOST_ROOT)/lib
    libs += boost_system boost_filesystem
    
    # 平台特定
    specific(prop:microsoft) {
        requires += debug
        Debug::cxxflags += /Zi /Od
        Release::cxxflags += /O2
    }
    
    specific(prop:gnu) {
        requires += debug
        Debug::cxxflags += -g -O0
        Release::cxxflags += -O2
    }
}
```

---

## 🔗 四、如何使用 common.mpb

### 4.1 在项目中继承

```mpc
# 库项目继承 common
project(mylib) : common {
    staticname = mylib
    Source_Files {
        mylib.cpp
    }
}

# 应用项目继承 common
project(myapp) : common {
    exename = myapp
    libs += mylib
    Source_Files {
        main.cpp
    }
}
```

### 4.2 在工作区中指定路径

```mpc
workspace(MyProject) {
    # 告诉 MPC 去哪里找 common.mpb
    cmdline += -include config
    
    mylib.mpc
    myapp.mpc
}
```

### 4.3 目录结构

```
myproject/
├── config/
│   └── common.mpb          # 公共配置
├── include/
│   └── ...
├── src/
│   ├── lib/
│   │   ├── mylib.cpp
│   │   └── mylib.mpc
│   └── app/
│       ├── main.cpp
│       └── myapp.mpc
└── workspace.mwc
```

---

## 📊 五、配置项速查表

| 配置项     | 用途           | 示例                           |
| ---------- | -------------- | ------------------------------ |
| `cxxflags` | C++ 编译器选项 | `cxxflags += -std=c++11 -Wall` |
| `cflags`   | C 编译器选项   | `cflags += -std=c99`           |
| `includes` | 头文件搜索路径 | `includes += include`          |
| `libs`     | 链接库         | `libs += pthread`              |
| `libpaths` | 库文件搜索路径 | `libpaths += /usr/local/lib`   |
| `macros`   | 预定义宏       | `macros += DEBUG`              |
| `version`  | 版本号         | `version = 1.0.0`              |
| `major`    | 主版本号       | `major = 1`                    |
| `minor`    | 次版本号       | `minor = 0`                    |
| `micro`    | 修订号         | `micro = 0`                    |
| `requires` | 需要的特性     | `requires += ssl`              |
| `avoids`   | 避免的特性     | `avoids += vc6`                |
| `after`    | 依赖顺序       | `after += other_lib`           |
| `recurse`  | 递归扫描       | `recurse = 1`                  |

---

## 🎯 六、最佳实践

### 6.1 保持简单

```mpc
# ✅ 好的做法：只放真正公共的配置
project(common) {
    cxxflags += -std=c++11 -Wall
    includes += include
    macros += _REENTRANT
}

# ❌ 不好的做法：放太多特定配置
project(common) {
    cxxflags += -std=c++11 -Wall -Wextra -Werror -pedantic ...
    includes += include include/extra include/third ...
    libs += pthread dl rt ssl crypto boost ...
    # 太多项目共享这些会导致过度耦合
}
```

### 6.2 分层配置

```mpc
# 基础配置
project(common_base) {
    cxxflags += -std=c++11
    includes += include
}

# 库专用配置
project(common_lib) : common_base {
    cxxflags += -fPIC
    macros += BUILDING_LIB
}

# 应用专用配置
project(common_app) : common_base {
    cxxflags += -DNDEBUG
}

# 各项目继承不同的基础配置
project(mylib) : common_lib { ... }
project(myapp) : common_app { ... }
```

### 6.3 使用条件配置

```mpc
project(common) {
    # 默认配置
    cxxflags += -Wall
    
    # 调试模式
    requires += debug
    Debug::cxxflags += -g -O0
    macros += _DEBUG
    
    # 发布模式
    requires += release
    Release::cxxflags += -O2
    macros += NDEBUG
}
```

---

## 💡 七、你的 Message Logger 项目的 common.mpb

根据你的项目需求，建议的 `common.mpb`：

```mpc
# config/common.mpb
project(common) {
    # 编译器选项
    cxxflags += -std=c++11
    cxxflags += -Wall
    cxxflags += -Wextra
    
    # 头文件路径（项目根目录下的 include）
    includes += include
    
    # 公共宏
    macros += _REENTRANT
    
    # 调试配置
    Debug::cxxflags += -g -O0 -DDEBUG
    Release::cxxflags += -O2 -DNDEBUG
    
    # 可选：平台特定
    specific(prop:gnu) {
        cxxflags += -fPIC
    }
}
```

然后各项目继承它：

```mpc
# src/core/logger.mpc
project(logger_core) : common {
    staticname = logger_core
    Source_Files { logger.cpp }
}

# src/file/file_logger.mpc
project(logger_file) : common {
    staticname = logger_file
    libs += logger_core
    Source_Files { file_logger.cpp }
}

# src/app/logger_app.mpc
project(logger_app) : common {
    exename = logger
    libs += logger_core logger_file
    Source_Files { main.cpp }
}
```

工作区指定 include 路径：

```mpc
workspace(MessageLogger) {
    cmdline += -include config
    src/core/logger.mpc
    src/file/file_logger.mpc
    src/app/logger_app.mpc
}
```