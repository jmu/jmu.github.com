---
layout: post
title: "Scala in Impatient 习题解答15  注解A2"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

1. 

    ```scala
    import org.junit.runner.{JUnitCore, RunWith}
    import org.junit.runners.JUnit4
    import org.junit.Test

    @RunWith(classOf[JUnit4])
    class MyTest {

      @Test
      def testNoParam() {
        assert(true)
      }

      @Test(timeout = 10)
      def testTimeout() {
        Thread.sleep(15)
      }

      @Test(expected = classOf[java.lang.Exception])
      def testExpected() {
        throw new java.lang.Exception("a expected exception")
      }

      @Test(timeout = 100, expected=classOf[java.lang.Exception])
      def testTimeoutAndExpected() {
        throw new java.lang.Exception("a expected exception")
      }
    }
    ```

  运行

        ```
        scalac -cp .:/home/junit-4.10.jar MyTest.scala
        java -cp .:/home/lib/junit-4.10.jar:/home/scala-2.9.2/lib/scala-library.jar org.junit.runner.JUnitCore MyTest
        ```

----
<div align="right">use Scala 2.11.1</div>
