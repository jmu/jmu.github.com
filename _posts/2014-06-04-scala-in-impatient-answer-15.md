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

        scalac -cp .:/home/junit-4.10.jar MyTest.scala
        java -cp .:/home/lib/junit-4.10.jar:/home/lib/scala-library.jar org.junit.runner.JUnitCore MyTest

2. 

    ```scala
    import scala.util.continuations.cpsParam

    @deprecated
    class AnnoClass[@specialized T](flag: Int, a: T) {
      @native var id = 1
      @unchecked
      def abc(@inline input: String @cpsParam) = 0
    }
    ```

3. 
`@param`: deprecatedName
`@field`: transient, volatile, BeanProperty, BooleanBeanProperty
`@getter`: deprecated
`@setter`: deprecated
`@beanGetter`: deprecated
`@beanSetter`: deprecated

4. 

    ```scala
    //test.scala
    import scala.annotation.varargs

    object Obj {
      @varargs def sum(a: Int*) = a.sum
    }
    ```
  Java:

    ```java
    public class JavaApp {
        public static void main(String[] args) {
            System.out.println("call scala Obj: " + Obj.sum(1,2,3,4,5));
        }
    }
    ```

  run

        scalac test.scala
        javac -cp .:/home/lib/scala-library.jar JavaApp.java
        java -cp .:/home/lib/scala-library.jar JavaApp 
        call scala Obj: 15

5. 

    ```scala
    import java.io.File
    import scala.io.Source

    object Obj {
      def mkString(f: File): String = Source.fromFile(f).mkString
    }
    ```
    ```java
    import java.io.File;

    public class JavaApp {
        public static void main(String[] args) {
            if (args == null || args.length != 1) {
                System.out.println("USAGE: JavaApp <file>");
                return;
            }
            System.out.println("call scala Obj: " + Obj.mkString(new File(args[0])));
        }
    }
    ```

         scalac test.scala 
         javac -cp .:/home/lib/scala-library.jar JavaApp.java
         java -cp .:/home/lib/scala-library.jar JavaApp /tmp/xxx.log

6. 

    ```scala
    import scala.compat.Platform.currentTime

    object Obj {
      @volatile var flag: Boolean = false

      def main(args: Array[String]) = {
        new Thread() {
          override def run() {
            Thread.sleep(1000)
            flag = true
            println("Flag set to true, over")
          }
        }.start

        new Thread() {
          override def run() {
            while (true) {
              if (flag) {
                println("Flag is true, confirmed")
                return
              } else Thread.sleep(100)
            }
          }
        }.start


      }
    }
    ```

to be continued.

----
<div align="right">use Scala 2.11.1</div>
