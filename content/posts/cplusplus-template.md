+++
date = '2026-03-11T18:35:45+08:00'
draft = false
title = 'C++模板学习笔记'
tags = [
  'C++', 'cpp', 'template'
]
+++
# C++模板

## 基本术语
- 函数模板(function template)：模板化的函数
- 类模板(class template)：模板化的类(class、struct、union)
- 变量模板(variable template)：模板化的变量
- 别名模板(alias template)：模板化的类型名

模板形参
- 类型模板形参(type template parameters)  
  如`template<typename T>`，形参表示的是使用模板时指定的类型
- 非类型模板形参(non-type template parameters)  
  如`template<size_t N>`，每一个形参必须有一个结构化类型，包括整型、浮点型(C++20)、指针类型等
- 模板模板形参(template template parameters)  
  如`template<typename K, template<typename> typename C>`，形参的类型是另一个模板

模板特化
- 部分特化(partial specialization)  
  只为部分模板形参提供替代实现
- (显式)完全特化((explicit)full specialization)  
  为所有模板形参提供特化实现

模板实例化
- 隐式实例化(implicit instantiation)  
  编译器由于代码中用到了模板而对其进行实例化，只会为用到的组合或实参进行实例化
- 显式实例化(explicit instantiation)  
  显式告诉编译器需要实例化哪些模板，即使代码中没有显式使用

## 变参模板
### 变参函数模板
```
template <typename T>
T min(T a, T b)
{
  return a < b ? a : b;
}

template <typename T, typename... Args>
T min(T a, Args... args)
{
  return min(a, min(args...));
}
```
- 模板形参包(template parameter pack)  
  在模板形参列表中指定形参包，如`typename... Args`，可用于类型模板形参、非类型模板形参、模板模板形参
- 函数形参包(function parameter pack)  
  在函数形参列表中指定形参包，如`Args... args`
- 形参包展开(paameter pack expansion)  
  在函数体中展开形参包，如`min(args...)`，其结果是一个由零个或多个值(或表达式)组成的都逗号分隔的列表

形参包中的实参数量可在编译器通过`sizeof...`运算符获取，此运算符返回一个类型为`std::size_t`的`constexpr`值。
```
template <typename T, typename... Args>
T sum(T a, Args... args)
{
  if constexpr (sizeof... (args) == 0)
    return a;
  else
    return a + sum(args...);
}
```

`sizeof...(args)`和`sizeof...(Args)`不等价，前者是用于形参包`args`上的`sizeof`运算符，后者是形参包`args`在`sizeof`运算符上的展开。
```
template <typename... Ts>
constexpr auto get_type_sizes()
{
  return std::array<std::size_t, sizeof...(Ts)>{sizeof(Ts)...};
}
auto sizes = get_type_sizes<short, int, long, long long>();
```
`sizeof...(Ts)`在编译期求值为`4`，而`sizeof(Ts)...`则展开为以逗号分隔的形参包：`sizeof(short), sizeof(int), sizeof(long), sizeof(long long)`

### 变参类模板
```
template <typename T, typename... Ts>
struct tuple
{
  tuple(T const& t, Ts const &... ts)
    : value(t), rest(ts...)
  {}

  constexpr int size() const { return 1 + rest.size(); }

  T            value;
  tuple<Ts...> rest;
};
template <typename T>
struct tuple<T>
{
  tuple(const T& t)
    : value(t)
  {}

  constexpr int size() const { return 1; }

  T value;
}
```

## 折叠表达式
折叠表达式是一种涉及形参包的表达式，在二元运算符上折叠(或减少)形参包中的元素。
```
template <typename T>
T sum(T a) { return a; }

template <typename T, typename... Args>
T sum(T a, Args... args)
{
  return a + sum(args...);
}
```
可通过折叠表达式简化为：
```
template <typename... T>
int sum(T... args)
{
  return (... + args);
}
```
`(... + args)`即为折叠表达式，圆括号不能省略，其在求值后变为`((((arg0 + arg1) + arg2) + ...) + argN)`。
总共有4种不同类型的折叠表达式：
|    折叠    |           语法        |                       展开                      |
|------------|-----------------------|-------------------------------------------------|
| 一元右折叠 | (pack op ...)         | (arg1 op (... op (argN-1 op argN)))             |
| 一元左折叠 | (... op pack)         | (((arg1 op arg2) op ...) op argN)               |
| 二元右折叠 | (pack op ... op init) | (arg1 op (... op (argN-1 op (argN op init))))   |
| 二元左折叠 | (init op ... op pack) | ((((init op arg1) op arg2) op ...) op argN)     |

`sum`中形参包`args`直接出现在折叠表达式中。形参包也可以成为表达式的一部分如下：
```
template <typename... T>
void printl(T... args)
{
  (..., (std::cout << args)) << '\n';
}

template <typename... T>
void printr(T... args)
{
  ((std::cout << args), ...) << '\n';
}

printl('d', 'o', 'g'); // (((std::cout<<'d'), std<<cout<<'o'), std::cout<<'g') => dog
printr('d', 'o', 'g'); // (std::cout<<'d', (std::cout<<'o', (std::cout<<'g'))) => dog
```
形参包`args`是`(std::cout << args)`表达式的一部分，这不是一个折叠表达式，`((std::cout << args), ...)`才是折叠表达式，是逗号运算符上的一元左折叠表达式。

## SFINAE 和 enable_if
SFINAE(Subsitution Failure Is Not An Error)。
