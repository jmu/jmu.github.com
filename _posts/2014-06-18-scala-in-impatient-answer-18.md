---
layout: post
title: "Scala in Impatient 习题解答18 高级类型L2"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

1. 

    ```scala
    class Bug {
      var steps = 0

      def move(step: Int): this.type = { steps += step; this }
      def turn(): this.type = { steps = 0; this}
      def show(): this.type = { print(steps + " "); this}
    }
    ```

        scala> val bug= new Bug
        bug: Bug = Bug@196ed85

        scala> bug.move(4).show().move(6).show().turn().move(5).show()
        4 10 5 res45: Bug = Bug@196ed85

2. 

    ```scala
    object around
    object then
    object show

    class Bug {
      var steps = 0

      def move(step: Int): this.type = { steps += step; this }
      def and(obj: then.type): this.type = this
      def and(obj: show.type): this.type = { print(steps + " "); this}
      def turn(obj: around.type): this.type = { steps = 0; this}
    }
    ```

3. 


----
<div align="right">use Scala 2.11.1</div>
