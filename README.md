---
description: 代码规范不仅仅是技术规范，更是项目管理规范。
layout: editorial
---

# ☝ C++ 代码规范

## 〇、前言

这篇文章主要是根据 Google 的 C++ 开源项目风格指南，结合笔者自身的理解所写，更多的是作为总结笔记。

> 这里是 Google C++ 风格指南的英文原版以及中文翻译链接：

<table data-view="cards"><thead><tr><th></th><th></th><th></th><th data-hidden data-card-cover data-type="files"></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><strong>Google C++ Style Guide</strong></td><td>英文原版</td><td></td><td><a href=".gitbook/assets/GoogleCppStyleGuide.png">GoogleCppStyleGuide.png</a></td><td><a href="https://google.github.io/styleguide/cppguide.html">https://google.github.io/styleguide/cppguide.html</a></td></tr><tr><td><strong>Google 开源项目风格指南</strong></td><td>中文版</td><td></td><td><a href=".gitbook/assets/Google开源风格.png">Google开源风格.png</a></td><td><a href="https://zh-google-styleguide.readthedocs.io/en/latest/">https://zh-google-styleguide.readthedocs.io/en/latest/</a></td></tr></tbody></table>

## 一、头文件

**每个 .cpp 文件都要有对应的同名 .h/.hpp 文件。**常见的例外是：单元测试，接入各平台的入口 main 源文件。

### 1.1 Self-Contained

头文件必须要自给自足（Self-contained），即头文件已经包含所有它所需的头文件，用户使用该头文件时不需要额外再 include，且无需提供任何特别的宏。

### 1.2 #pragma once

头文件都应该使用 `#pragma once` 来避免被多重包含，相较于 `#define` 的方式更省心省力。

### 1.3 前置声明

当一个文件里仅仅是引用到一个类型，而<mark style="color:purple;">**没有访问其中的具体变量或函数**</mark>，就可以用前置声明来提升编译速度。

### 1.4 内联函数

只有当函数只有 10 行甚至更少时才将其定义为内联函数，通常是一些简单的 get/set、转换接口。

此外，不要内联析构函数，因为有隐含的成员和基类析构函数被调用！包含循环和 switch 的函数不建议内联，虚函数和递归函数不建议内联。

### 1.5 #include 的规范

\#include 应按以下分类按顺序书写：

1. C 系统文件
2. C++ 系统文件
3. 其他库的 .h 文件
4. 本项目的 .h 文件

不同分类之间同空行隔开，同分类下的头文件按照字典序进行排序。若有条件编译的头文件，则放到最后。实例：

```cpp
// C系统库
#include <cmath>
#include <cstdio>

// C++系统库
#include <hash_map>
#include <vector>

// 其他库
#include "SDL2.h"
#include "spdlog/spdlog.h"

// 本项目库
#include "base/basictypes.h"
#include "base/commandlineflags.h"
#include "foo/public/bar.h"

// 条件编译
#ifdef LANG_CXX11
#include <initializer_list>
#endif  // LANG_CXX11
```

