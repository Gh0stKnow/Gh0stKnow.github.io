---
title: Swift (?)可选类型和 (!)强制解包
permalink: swiftzhong-de-ke-xuan-lei-xing-he-qiang-zhi-jie-xi
id: 29
updated: '2016-07-21 03:05:09'
date: 2016-07-13 15:00:33
tags: swift
---



很多Swift初学者在学习过程中对问号和感叹号比较迷惑，
Swift中**问号**表示这是一个可选类型。
翻译：某个常量或者变量可能是一个类型，也可能什么都没有，**我不确定它是否真的会有值，也许会是nil**。

比如：

> let number1 = “123”
>
> let number2 = number1.toInt()
>
> //number2是Int?类型 或者叫optional Int

number2，可能包含Int值，也可能什么都没有（那就是nil）

当我们通过if语句判断，number2确实有值时，可以使用**感叹号**来表示。

翻译：**我确定这个可选类型的值内有正确的内容，请使用它**。

> if number2 != nil {
>
>     println(number2!)

感叹号，被称为可选值的强制解析（无论如何请一定要使用这个值，我保证这里面肯定有东西）

然而在使用强制解析的时候，一定要确保包中的内容确实不会为 nil，**否则会报错**。

