---
layout: post
title: Haskell中的多态之参数多态
permalink: haskell-params-polymorphism
author: cailin
---

## 参数多态

> 参数多态在程序设计语言与类型论中是指声明与定义函数、复合类型、变量时不指定其具体的类型，而把这部分类型作为参数使用，使得该定义对各种具体类型都适用

例如Haskell的`id`函数：
```Haskell
id :: forall a. a -> a
id x = x
```
`id`函数的签名可以解释为：对于任意类型`a`，如果`x`的类型是`a`，那么`id x`都是合法的，所以`id "abc"`, `id 1`, `id True`等调用都是合法的

而像`not :: Bool -> Bool` 这样的函数则只能以Bool类型的值为参数

## RankN-Type

再考虑以下的函数

```Haskell
foo :: forall a. (a -> a) ->  Bool
foo f = f True
```
是不能编译通过的，因为按照`foo`的签名，对于任意类型`a`，如果`f`的类型是`a -> a`，都应该保证`foo f`是合法的，但如果取`a`为`Int`，`f`的类型就为`Int -> Int`,但是`f True`显然就不合法了。另一种更简单的解释是：这里`f`的类型是`a -> a`，但是你却给`f`传了个`Bool`类型的值，当然就类型不匹配了，GHC的报错信息就是这样的，这说明，`foo`是一个多态函数，而`f`却不是。

解决这个问题的方法是开启GHC的`RankNTypes`扩展并写成下面的样子：

```Haskell
foo' :: (forall a. a -> a) -> Bool
foo' f = f True
```
我们只是加了一层括号，但是签名的含义发生了很大的变化——如果`f`的类型为`forall a. a -> a`，那么`foo' f`是合法的，由于`id`的类型为`forall a. a -> a`，因此`foo' id`是合法的。可以看出，现在我们要求`f`是一个多态函数，当然`foo'`也还是一个多态函数，但这种多态已经和`id`不是一回事了。

像`id`这样类型参数只是简单的类型变量（而不是参数多态类型）的参数多态类型被称作是`Rank-1`的，而像`foo'`这样其类型参数是一个`Rank-1`的参数多态类型被称为是`Rank-2`的，由此可知，如果一个类型的类型参数中最高`Rank`为`N`，那么这个类型就是`Rank-N+1`的。

可以将参数多态类型理解类比为函数，它们有如下的对应关系：

| 参数多态类型 | 函数 |
| ----- | ----- |
| 类型参数 | 函数参数 |
| 类型参数为简单类型变量(a, b, c..) | 函数参数类型为简单类型(a, b, c..) |
| 类型参数为参数多态类型 | 函数参数为函数 |
| 参数多态类型是Rank-N的 | 函数的Rank为N |
| 类型参数为Rank-N的，那么此参数多态类型为Rank-N+1的 | 函数参数的Rank为N，那么此函数的Rank为N+1 |