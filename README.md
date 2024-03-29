---
description: 代码规范不仅仅是技术规范，更是项目管理规范。
---

# ☝ C++ 代码规范

## 〇、前言

这篇文章主要是根据 Google 的 C++ 开源项目风格指南，结合笔者自身的理解所写，更多的是作为总结笔记。

> 这里是 Google C++ 风格指南的英文原版以及中文翻译链接：

<table data-view="cards"><thead><tr><th></th><th></th><th></th><th data-hidden data-card-cover data-type="files"></th><th data-hidden data-card-target data-type="content-ref"></th></tr></thead><tbody><tr><td><strong>Google C++ Style Guide</strong></td><td>英文原版</td><td></td><td><a href=".gitbook/assets/GoogleCppStyleGuide.png">GoogleCppStyleGuide.png</a></td><td><a href="https://google.github.io/styleguide/cppguide.html">https://google.github.io/styleguide/cppguide.html</a></td></tr><tr><td><strong>Google 开源项目风格指南</strong></td><td>中文版</td><td></td><td><a href=".gitbook/assets/Google开源风格.png">Google开源风格.png</a></td><td><a href="https://zh-google-styleguide.readthedocs.io/en/latest/">https://zh-google-styleguide.readthedocs.io/en/latest/</a></td></tr></tbody></table>



## 一、头文件

**每个 .cpp 文件都要有对应的同名 .h/.hpp 文件。**常见的例外是：单元测试，接入各平台的入口 main 源文件。

### 1.1 Self-Contained

头文件必须要自给自足 (Self-contained)，即头文件已经包含所有它所需的头文件，用户使用该头文件时不需要额外再 include，且无需提供任何特别的宏。

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
5. 条件编译投文件

不同分类之间同空行隔开，同分类下的头文件按照字典序进行排序。实例：

{% code lineNumbers="true" %}
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
{% endcode %}



## 二、作用域

### 2.1 命名空间

* 任何类型都需要定义在某个命名空间下。命名空间的作用就是为了避免命名冲突，在根命名空间下的类型会增加命名冲突的风险。
* 定义在命名空间下的类型要加缩进，且在命名空间闭括号后注释。
* 禁止使用 using 引入整个命名空间标识符。

{% code lineNumbers="true" %}
```cpp
// 禁止的操作！会污染命名空间
using namespace std;
```
{% endcode %}

* 禁止使用内联命名空间。内联命名空间会扰乱命名空间的层级关系。

{% code lineNumbers="true" %}
```cpp
// 禁止内联命名空间！
// 以下代码会导致X::Y::foo()与X::foo()等价
namespace X
{
    inline namespace Y
    {
        void foo();
    }
}
```
{% endcode %}

### 2.2 静态变量

对于不需要在其他源文件中使用的变量/标识符，鼓励在 .c/.cpp 文件中定义静态变量，但是不要在 .h 中定义。

{% hint style="info" %}
在同一个编译单元内，静态初始化总是先于动态初始化，初始化顺序按照声明顺序，销毁则相反。不同编译单元之间的初始化顺序属于未明确行为 (Unspecified Behaviour)。
{% endhint %}

### 2.3 非成员函数 & 静态成员函数

* 非成员函数不应依赖于外部变量，应尽量置于某个命名空间内。
* 静态成员函数应当和类的实例或静态数据紧密相关。
* 相比单纯为了封装若干不共享任何静态数据的静态成员函数而创建类，不如使用命名空间：

{% code lineNumbers="true" %}
```cpp
// 推荐！
namespace myproject
{
    namespace foo_bar
    {
        void Function1();
        void Function2();
    }  // namespace foo_bar
}  // namespace myproject

// 不推荐！
namespace myproject
{
    class FooBar
    {
        void Function1();
        void Function2();
    }
}  // namespace myproject
```
{% endcode %}

### 2.4 局部变量

将函数变量尽可能置于最小作用域内，并在变量声明时进行初始化。

如果变量是一个对象，则要避免频繁的构造和析构：

{% code lineNumbers="true" %}
```cpp
Foo f;  // 构造函数和析构函数只调用1次，推荐！
for (int i = 0; i < 1000000; ++i)
{
    f.DoSomething(i);
}
```
{% endcode %}



## 三、类

### 3.1 构造函数

不要在构造函数中调用虚函数。若在构造函数中调用了虚函数，它是不会重定向到子类实现的。e.g.

{% code lineNumbers="true" fullWidth="false" %}
```cpp
#include <iostream>

class A
{
public:
    A() { func(); }
    virtual void func() { std::cout << "A::func\n"; }
};

class B : public A
{
public:
    B() { func(); }
    virtual void func() override { std::cout << "B::func\n"; }
};

int main()
{
    auto b = new B();
    std::cout << "--------\n";
    delete b;
    return 0;
}

// 输出：
// A::func
// B::func
// --------
// B::func
// A::func
```
{% endcode %}

在创建子类对象时，先后调用了 `A::func` 和 `B::func`，因为构造时先进入父类构造函数，这时对象类型是 `A`，然后进入子类构造函数，这时对象类型是 `B`。析构过程则相反，先调用 `~B()` 再调用 `~A()`。

在 C++ 的构造和析构函数中均<mark style="color:purple;">**无法正确调用虚函数的重写版本**</mark>，但是 C# 可以重定向到子类实现的。

### 3.2 结构体 vs 类

在 C++ 中 `struct` 和 `class` 关键字几乎含义一样。在实践中，结构体偏向于数据，类偏向于功能。若成员只是数据和对数据的简单操作，定义成 `struct`；若包含逻辑代码，定义成 `class`。

### 3.3 继承

继承一定用 public，数据成员尽量使用 private，重载的虚函数用 virtual 与 override/final 修饰。

多重继承只有在一种情况下才使用：除第一个基类外，其余均为接口类。

### 3.4 声明顺序

将相似声明放在一起，可用 `#pragma region/endregion` 块整理，并且按访问权限顺序声明： public → protected → private。

{% code lineNumbers="true" %}
```cpp
class PlayerState
{
public:
    PlayerState();

    #pragma region Getter/ Setter

    inline float GetHealth() const { return health; }
    inline float GetMana() const { return mana; }

    #pragma endregion

private:
    float health;
    float mana;
};
```
{% endcode %}



## 四、函数

### 4.1 简短函数

更短的函数具有更强的可读性，如果一个函数超过了 50 行，或者一个屏幕的高度显示不全，你就需要考虑是否能把它拆成更简短的函数了。

### 4.2 引用参数

所有的引用参数必须加上 `const`，若参数需要更改，则用指针。

### 4.3 缺省参数

<mark style="color:red;">**不准在虚函数中使用缺省参数！**</mark>

### 4.4 返回类型后置语法

在一般的函数中正常的前置语法，而在 Lambda 表达式中使用后置语法，能增强可读性，在模板中使用后置语法，能简化书写。

<pre class="language-cpp" data-line-numbers><code class="lang-cpp"><strong>transform(v.begin(), v.end(), v.begin(), [](int i) -> int
</strong><strong>{
</strong><strong>    return i &#x3C; 0 ? -i : i;
</strong><strong>});
</strong><strong>
</strong><strong>template&#x3C;class T, class U>
</strong>auto add(T t, U u) -> decltype(t + u);
</code></pre>
