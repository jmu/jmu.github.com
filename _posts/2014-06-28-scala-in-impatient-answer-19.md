---
layout: post
title: "Scala in Impatient 习题解答19 解析A3"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

1. 为算数表达式求值器添加`/`和`%`操作符
  `*`,`/`,`%`是属于同一个优先级的，所以直接加到`*`的后面。

    ```scala
    import scala.util.parsing.combinator.RegexParsers

    class ExprParser extends RegexParsers {
      val number = "[0-9]+".r

      def expr: Parser[Double] = term ~ rep(("+" | "-") ~ term ^^ {
        case "+" ~ t => t
        case "-" ~ t => -t
      }) ^^ {case t ~ r => t + r.sum }

      def term: Parser[Double] = factor ~ rep(("*" | "/" | "%") ~ factor) ^^ {
        case f ~ list => list.foldLeft(f)((a, b) => b._1 match {
          case "*" => a * b._2
          case "/" => a / b._2
          case "%" => a % b._2
        })
      }
      def factor: Parser[Double] = number ^^ {_.toDouble} | "(" ~> expr <~ ")"

    }

    object ParserMain extends App {
      val parser = new ExprParser()
      val result = parser.parseAll(parser.expr,"3-4 * 5 + 3 / (8 % 6)")
      if (result.successful)
        println(result.get)
    }
    ```

2. 

    ```scala
    import scala.util.parsing.combinator.RegexParsers

    class ExprParser extends RegexParsers {
      val number = "[0-9]+".r


      def expr: Parser[Double] = term ~ rep(("+" | "-") ~ term ^^ {
        case "+" ~ t => t
        case "-" ~ t => -t
      }) ^^ {case t ~ r => t + r.sum }

      def term: Parser[Double] = factor ~ rep(("*" | "/" | "%") ~ factor) ^^ {
        case f ~ list => list.foldLeft(f)((a, b) => b._1 match {
          case "*" => a * b._2
          case "/" => a / b._2
          case "%" => a % b._2
        })
      }
      def factor: Parser[Double] = inner ~ rep("^" ~> inner) ^^ {
         case e ~ Nil => e
         case e ~ list => math.pow(e, list.reduceRight(math.pow(_, _)))
      }
      def inner: Parser[Double] = number ^^ {_.toDouble} | "(" ~> expr <~ ")"

    }

    object ParserMain extends App {
      val parser = new ExprParser()
      val result = parser.parseAll(parser.expr,"3-4 * 5 + 3 / (8 % 6) + 4 ^ 2 ^ 3")
      if (result.successful)
        println(result.get)
    }
    ```

3. 

    ```scala 
    import scala.util.parsing.combinator.RegexParsers

    class ListParser extends RegexParsers {
      val number = "(-?[0-9])+".r

      def expr: Parser[List[Int]] = repsep(number,",") ^^ {
        case list => list.map(_.toInt)
      }

    }

    object LPMain extends App {
      val parser = new ListParser()
      val result = parser.parseAll(parser.expr,"1,23,-79")
      if (result.successful)
        println(result.get)
    }
    ```

4. 

    ```scala
    import java.util.{Date, Calendar}

    import scala.util.parsing.combinator.RegexParsers

    class ISODateParser extends RegexParsers {
      val number = "[0-9]+".r

      val hyphen = "-"
      val cln = ":"

      def expr: Parser[Date] = date ~ ("T" ~> time) ^^ {
        case d ~ t =>
          t.set(d.get(Calendar.YEAR), d.get(Calendar.MONTH), d.get(Calendar.DATE))
          t.getTime
      }

      def date: Parser[Calendar] = (number <~ hyphen) ~ (number <~ hyphen) ~ number ^^ {
        case y ~ m ~ d =>
          val cal: Calendar = Calendar.getInstance
          cal.set(y.toInt, m.toInt, d.toInt)
          cal
      }

      def time: Parser[Calendar] = (number <~ cln) ~ (number <~ cln) ~ number ^^ {
        case h ~ m ~ s =>
          val cal: Calendar = Calendar.getInstance
          cal.set(Calendar.HOUR_OF_DAY, h.toInt)
          cal.set(Calendar.MINUTE, m.toInt)
          cal.set(Calendar.SECOND, s.toInt)
          cal
      }
    }

    object LPMain extends App {
      val parser = new ISODateParser
      val result = parser.parseAll(parser.expr, "2014-01-01T21:53:00")
      if (result.successful)
        println(result.get)
    }
    ```

5. 
6. 
7. 
8. 
9. 
10. 

----
<div align="right">use Scala 2.11.1</div>

