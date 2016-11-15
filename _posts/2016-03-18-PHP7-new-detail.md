---
layout: post
title: PHP7新特性总结
description: "描述PHP7新特性。PHP7新特性总结"
modified: 2016-03-18
tags: [php]
categories: [intro]
---


	PHP7版本相较于老版本，加了大量新特性，同时性能得到了显著提升：是PHP5.6性能的2倍，在wordpress表现上超过了HHVM。下面来总结下PHP7新增的特性。
# 一、新增特性

## 1、标量类型声明
	现在可以使用下列类型参数（无论用强制模式还是严格模式）：字符串(string),整数 (int), 浮点数 (float), 以及布尔值 (bool)。它们扩充了PHP5中引入的其他类型：类名，接口，数组和回调类型。其有两种