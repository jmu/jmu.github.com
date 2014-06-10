---
layout: post
title: "Scala in Impatient 习题解答16 XML处理A2"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

1. `<fred/>(0)`得到什么？`<fred/>(0)(0)`呢？为什么？

  `<fred/>`是一个Elem，只有一个子元素Node，`<fred/>(0)`就会返回这个Node
  而Node继承了`Seq[Node]`，它实际上是一个大小是1的数组。那么`<fred/>(0)(0)`就会返回它本身。
  以下可以证明

```scala
scala> <fred/>(0) == <fred/>(0)(0)
res17: Boolean = true
```

2. 如下代码的值是什么？
<ul>
  <li>Opening bracket: [</li>
  <li>Closing bracket: ]</li>
  <li>Opening brace: {</li>
  <li>Opening brace: }</li>
</ul>
你如何修复它？

<ul>
  <li>Opening bracket: <![CDATA[[]]></li>
  <li>Closing bracket: <![CDATA[]]]></li>
  <li>Opening brace: {{</li>
  <li>Opening brace: }}</li>
</ul>

3. 

----
<div align="right">use Scala 2.11.1</div>
