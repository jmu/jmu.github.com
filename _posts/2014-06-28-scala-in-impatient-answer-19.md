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

    ```scala
    import scala.util.parsing.combinator.{PackratParsers, RegexParsers}
    import scala.util.parsing.input.CharSequenceReader

    class Expr

    case class Number(value: Int) extends Expr

    case class Operator(op: String, left: Expr, right: Expr) extends Expr

    class ExprParser extends RegexParsers with PackratParsers {
      val wholeNumber = "[0-9]+".r

      lazy val expr: PackratParser[Expr] = (opt(expr ~ ("+" | "-")) ~ term) ^^ {
        case None ~ a => a
        case Some(b ~ op) ~ a => Operator(op, b, a)
      }

      lazy val term: PackratParser[Expr] = (factor ~ opt("*" ~> term)) ^^ {
        case a ~ None => a
        case a ~ Some(b) => Operator("*", a, b)
      }

      lazy val factor: PackratParser[Expr] = wholeNumber ^^ (n => Number(n.toInt)) |
        "(" ~> expr <~ ")"

      def parseAll[T](p: Parser[T], input: String) =
        phrase(p)(new PackratReader(new CharSequenceReader(input)))
    }

    object ParserMain extends App {
      val parser = new ExprParser()
      val result = parser.parseAll(parser.expr, "3-4 -5 ")
      if (result.successful)
        println(result.get)
    }
    ```

7. 

    ```scala
    import scala.util.parsing.combinator.RegexParsers

    class ExprParser extends RegexParsers {
      val number = "[0-9]+".r

      def expr: Parser[Double] = term ~ rep(("+" | "-") ~ term) ^^ {
        case t ~ r => (t /: r)((a, b) => b._1 match {
          case "+" => a + b._2
          case "-" => a - b._2
        })
      }

      def term: Parser[Double] = number ^^ { _.toDouble } | "(" ~> expr <~ ")"
    }

    object ParserMain extends App {
      val parser = new ExprParser()
      val result = parser.parseAll(parser.expr, "3-4 -5")
      if (result.successful)
        println(result.get)
    }
    ```

8. 

9. 
10. 

----
<div align="right">use Scala 2.11.1</div>

