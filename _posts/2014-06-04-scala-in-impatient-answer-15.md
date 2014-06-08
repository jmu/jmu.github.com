---
layout: post
title: "Scala in Impatient 习题解答15  注解A2"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

1. 编写四个JUnit测试用例，分别使用带或不带某个参数的@Test
注解。用JUnit执行这些测试

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

2. 创建一个类的示例，展示注解可以出现的所有位置。用@deprecated作为你的示例注解

    ```scala
    import scala.util.continuations.cpsParam

    @deprecated
    class AnnoClass[@specialized T](flag: Int, a: T) {
      @native var id = 1
      @unchecked
      def abc(@inline input: String @cpsParam) = 0
    }
    ```

3. Scala类库中的哪些注解用到了元注解
@param,@field,@getter,@setter,@beanGetter或@beanSetter?

        `@param`: deprecatedName
        `@field`: transient, volatile, BeanProperty, BooleanBeanProperty
        `@getter`: deprecated
        `@setter`: deprecated
        `@beanGetter`: deprecated
        `@beanSetter`: deprecated

4. 编写一个Scala方法sum,带有可变长度的整型参数，返回所有参数之和。从Java调用该方法。

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

5. 编写一个返回包含某文件所有行的字符串的方法。从Java调用该方法。

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

6. 编写一个Scala对象，该对象带有一个易失(volatile)的Boolean字段。让某一个线程睡眠一段时间，之后将该字段设为true，打印消息，然后退出。而另一个线程不停的检查该字段是否为 true。如果是，它将打印一个消息并退出。如果不是，则它将短暂睡眠，然后重试。如果变量不是易失的，会发生什么？

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

  没有区别

7. 给出一个示例，展示如果方法可被重写，则尾递归优化为非法

    ```scala
    import scala.annotation.tailrec

    class Util {
      @tailrec
      def printStr(s: String) {
        if (s.length > 0) {
          println(s.head)
          printStr(s.tail) 
        }
      }
    }
    ```

  错误消息如下,改成final可以修正这个问题。

        <console>:13: error: could not optimize @tailrec annotated method
        printStr: it is neither private nor final so can be overridden
                 def printStr(s: String) {
                     ^

8. 将allDifferent方法添加到对象，编译并检查字节码。
@specialized注解产生了哪些方法?

    ```scala
    object MyObj {
      def allDifferent[@specialized T] (x: T, y: T, z: T) = {}
    }
    ```

  执行结果如下，生成了9种

        Compiled from "fun.scala"
        public final class MyObj$ {
          public static final MyObj$ MODULE$;
          public static {};
          public <T extends java/lang/Object> void allDifferent(T, T, T);
          public void allDifferent$mZc$sp(boolean, boolean, boolean);
          public void allDifferent$mBc$sp(byte, byte, byte);
          public void allDifferent$mCc$sp(char, char, char);
          public void allDifferent$mDc$sp(double, double, double);
          public void allDifferent$mFc$sp(float, float, float);
          public void allDifferent$mIc$sp(int, int, int);
          public void allDifferent$mJc$sp(long, long, long);
          public void allDifferent$mSc$sp(short, short, short);
          public void allDifferent$mVc$sp(scala.runtime.BoxedUnit, scala.runtime.BoxedUnit, scala.runtime.BoxedUnit);
        }

9. Range.foreach方法被注解为@specialized(Unit)。为什么？ 

        javap -classpath /home/jmu/lib/scala-library.jar scala.collection.immutable.Range

  执行javap的结果

        public final <U extends java/lang/Object> void foreach(scala.Function1<java.lang.Object, U>);

  使用`@specialized[Unit]`使Range只生成返回Unit的foreach函数.这样
  做的原因恐怕与scala中的所有表达式都返回Unit有关，比如`for (a <- 1 to 5)
  print _`应该返回Unit。


        @annotation.implicitNotFound(msg = "No implicit view available from ${T1} =>
        ${R}.")
        trait Function1[@specialized(scala.Int, scala.Long, scala.Float,
        scala.Double) -T1, @specialized(scala.Unit, scala.Boolean, scala.Int,
        scala.Float, scala.Long, scala.Double) +R] extends AnyRef { self =>

  Function1的返回值去掉了Char, Byte, not sure for why

10. 添加assert(n >= 0)到 factorial方法。在启用断言的情况下编译并校验factorial(-1)会
抛异常。在禁用断言的情况下编译。会发生什么？用javap检查该断言调用 

    ```scala
    object Test {
      def factorial(n: Int): Int = {
        assert(n > 0)
        n
      }

      def main(args: Array[String]) {
        factorial(-1)
      }
    }
    ```

  禁用断言下编译后，就不会再报assertion failed的异常了。
  这次我使用了sbt来进行编译。sbt的确如人所说，入手比较难用。
  在`sbt>`交互界面下设置禁用断言的option

        set scalacOptions in ThisBuild ++= Seq("-Xelide-below", "2001")

  再次compile后，断言不会被编译到class里。使用javap验证一下,发现 assert的调用的确没有了

----
<div align="right">use Scala 2.11.1</div>
