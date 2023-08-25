---
title: C++模板技巧——SIFNAE（从C++11到C++20）
tags:
  - 模板元编程
  - SIFNAE
categories:
  - CPP
  - template
abbrlink: 6f5fcdb3
date: 2023-08-24 19:16:13
updated: 2023-08-24 19:16:13
---

**SFINAE** 即“替换失败不是错误” (Substitution Failure Is Not An Error)。

在函数模板的重载决议中会应用此规则：当模板形参在替换成显式指定的类型或推导出的类型失败时，从重载集中丢弃这个特化，而非导致编译失败。

此特性被用于模板元编程。

<!-- more -->

## 那么**SFINAE**有什么用呢？

先引入一个问题：**如何在编译期检测一个类中是否存在某个函数？**

我们可以写出这样的代码：

``` cpp
template<typename T>
struct has_functor {
    struct always_functor {
        char dummy;
    };
    struct never_functor {
        char dummy[2];
    };

    template <typename U, void (U::*)()>
    struct SFINAE {};

    template<typename U>
    static always_functor check(SFINAE<U, &U::func>*);

    template<typename>
    static never_functor check(...);

    static constexpr bool value = sizeof(check<T>(nullptr)) == sizeof(always_functor);
};

template<typename T>
inline static constexpr bool has_functor_v = has_functor<T>::value;
```
测试一下

``` cpp
struct A { void func() { } };
struct B { };
int main() {
    std::cout << has_functor_v<A> << '\n';  // 1
    std::cout << has_functor_v<B> << '\n';  // 0
}
``` 
解释一下上面的代码：
- 首先定义两个结构 always_functor 和 never_functor， 内容并不重要，只需要大小不一样；
- 然后定义结构 SFINAE，内容同样不重要，但模板第二个参数是第一个参数的成员函数，无参数类型，返回类型是 void；
- 定义两个 check 函数，第一个要求参数为 SFINAE，返回 alwys_functorm，第二个可以接受任何参数，返回 never_functor；
- 最后定义 value，值取决于 nullptr 与哪一个 check() 匹配成功，因为 check(...) 的参数 ... 的优先级很低，如果另一个存在的话将会优先匹配另一个。

只需要调用 has_functor<T>::value 或 has_functor_v<T> 即可判断类中是否存在函数 void func()。


## std::void_t

std::void_t 是 C++17 引入的一个模板，它的定义如下：

``` cpp
template<typename...> using void_t = void;
```

它的定义非常简单，功能也非常简单，该类型模板会把任意类型映射到 void。而在这个过程中，编译器会检查转换类型的有效性。

基于 std::void， 我们可以利用 decltype、std::declval 大大简化代码。

``` cpp
template<typename T, typename = std::void_t<>>
struct has_functor : std::false_type {};

template<typename T>
struct has_functor<T, std::void_t<decltype(std::declval<T>().func())>> : std::true_type {};

template<typename T>
inline static constexpr bool has_functor_v = has_functor<T>::value;
```
std::false_type 和 std::true_type 是 std::integral_constant 的布尔值定义，分别是 std::integral_constant<bool,false> 和 std::integral_constant<bool, true>；

std::integral_constant 的定义如下：
``` cpp
template<typename T, T v>
struct integral_constant
{
    static constexpr T value = v;
    typedef T value_type;
    typedef integral_constant<T, v> type;
};
```
利用 std::integral_constant 可以很方便的取出类型与值。

std::declval<T>() 的作用是返回一个 T 对象的伪造实例，同时又具有右值引用参考。

有了这个伪造实例，就可以在该对象上施加操作，例如调用成员函数什么的，但既然是虚拟的，就不会真的存在这么个临时对象。

我们常常并不真的直接需要 declval 求值求得的伪实例，更多的是需要借助于这个伪实例来求取到相应的类型描述，也就是 T。所以一般情况下 declval 之外往往包围着 decltype 计算，设法拿到 T 才是我们的真实目的

那么上面的代码就很好理解了，第二个 has_functor 模板定义是一个类模板偏特化，当满足第二个 has_functor 时编译器会优先选择第二个特化模板。

这段代码与之前的功能基本一样，唯一的区别就是我们现在不需要关心 func() 的返回值类型了。

## 概念与约束

在 C++20 中提出了概念和约束到标准中。

### requires

先看看约束，requires 是一个关键字。可以直接在模板函数中进行使用。

来看一个例子：

``` cpp
template<typename T>
void foo(T x) {
    std::cout << "Other " << x << '\n';
}

template<typename T>
requires std::is_integral_v<T>
void foo(T x) {
    std::cout << "Int " << x << '\n';
}

template<typename T>
requires std::is_same_v<T, double>
void foo(T x) {
    std::cout << "Double " << x << '\n';
}

int main() {
    foo("abc");
    foo(1.0);
    foo(2);
}
```
输出为

```
Other abc
Double 1
Int 2
```

### concept

概念（concept） 是要求的具名集合。

concept 定义拥有以下形式

``` cpp
template < 模板形参列表 >
concept 概念名 属性(可选) = 约束表达式;
```

可以通过该关键字定义一个基于模板参数的约束条件。再将该约束条件运用到具体的模板函数中。同时可以复用这个约束条件。

``` cpp
template<typename T>
concept IterType = std::is_integral_v<T>;
template<IterType T>
T foo(T a, T b) {
    return a + b;
}
```

上面这个例子中定义 IterType 必须是整型。

### requires 表达式

requires 表达式用来产生一个描述约束的 bool 类型的纯右值表达式。

语法有两种

``` cpp
requires { 要求序列 }		
requires ( 参数列表(可选) ) { 要求序列 }		
```

利用它我们可以更好的去定义 concept

``` cpp
// 约束 MulType 重载了 *
template<typename T>
concept MulType = requires(T a, T b) {
    a * b;
};
template<MulType T>
T Mul(T a, T b) {
    return a * b;
}
```

如果不考虑复用，还可以这样子写

``` cpp
template<typename T>
requires requires(T a, T b) {
    a * b;
}
T Mul(T a, T b) {
    return a * b;
}
```

这里通过 requires 表达式定义了一个临时的匿名 concept

更多的语法可以参考[约束与概念 (C++20 起)](https://zh.cppreference.com/w/cpp/language/constraints)，这里只是简单交代以下用法。

## C++20 中的"SFINAE"

回到文中最开始的问题，我们使用 concept 和 requires 可以非常简单的实现

``` cpp
template<typename T>
concept has_functor = requires {
	T::func();
};
struct A { void func() {} };
struct B { };
int main() {
    std::cout << has_functor<A> << '\n';    // 1
    std::cout << has_functor<B> << '\n';    // 0
}
```
那如果我并不关心 func 的参数是什么呢?

假如有这么一个类

``` cpp
struct C { void func(int x) {}};
```
使用之前的代码它会返回 false，如果期望它返回 true 呢？

那只需要把 requires 表达式里的括号取消就行了：

``` cpp
template<typename T>
concept has_functor = requires {
	T::func;
};
```

但这样写又会导致另一个问题，如果一个类中有 func 同名变量也会返回 true

``` cpp
// has_functor<D> == true;
struct D { int func; }; 

```

同时还会有 overload 的问题

``` cpp
// has_functor<E> == false;
struct E {
    void func() {}
    void func(int x) {}
}
```

所以刚好的检测手段还是通过检测函数调用表达式的合法性，而非检测成员函数的名称。

那么我们可以改一改 has_functor

``` cpp
template<typename T, typename... Args>
concept has_functor = requires(T&& t, Args&&... args) {
    std::forward<T>(t).func(std::forward<Args>(args)...);
};
```

然后调用时加上所需要判断的成员函数参数变量

``` cpp

template<typename T, typename... Args>
concept has_functor = requires(T&& t, Args&&... args) {
    std::forward<T>(t).func(std::forward<Args>(args)...);

};
struct A { void func() { } };
struct B { };
struct C { void func(int i){ } };
struct D { int func; };
struct E { void func(){} void func(int x) {} };

int main() {
    std::cout << has_functor<A> << '\n';        // 1
    std::cout << has_functor<B> << '\n';        // 0
    std::cout << has_functor<C> << '\n';        // 0
    std::cout << has_functor<C, int> << '\n';   // 1
    std::cout << has_functor<D> << '\n';        // 0
    std::cout << has_functor<E> << '\n';        // 1
    std::cout << has_functor<E, int> << '\n';   // 1

}
```

这样基本能解决绝大部分问题了。

---



<!-- Q.E.D. -->
