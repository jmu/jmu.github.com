---
layout: post
title: "Maven Malformed \\uxxxx encoding"
description: ""
category: Hint
tags: [Maven]
---
{% include JB/setup %}

Maven编译突然发生异常

    maven org apache maven InternalErrorException Malformed \uxxxx encoding

在遍寻不到到底哪里出现斜杠后，发现换一下m2的目录就可以了。

  原来意外情况，如断电，切换操作系统时
损坏了Maven在reposity目录下的文件。
 清除后重新编译即可。
