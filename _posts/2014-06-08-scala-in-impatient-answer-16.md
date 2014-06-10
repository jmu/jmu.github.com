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

3. 比对
`<li>Fred</li> match { case <li>{Text(t)}</li> => t }`
和
`<li>{"Fred"}</li> match { case <li>{Text(t)}</li>`
为什么它们的行为不同？

        scala><li>Fred</li> match { case <li>{Text(t)}</li> => t }
        res31: String = Fred

        scala> <li>{"Fred"}</li> match { case <li>{Text(t)}</li> => t }
        scala.MatchError: <li>Fred</li> (of class scala.xml.Elem)
          ... 32 elided

  `<li>{"Fred"}</li>`的内嵌的Fred不会生成Text对象，而是生成`Atom[String]`,所以无法匹配。
  修改成以下就可以匹配了

    ```scala
    <li>{"Fred"}</li> match { case <li>{t: Atom[String] @unchecked}</li> => t }
    ```

4. 
创建16.xml，内容如下

        <root>
          <a><img src="1.jpg"/></a>
          <a><img src="1.jpg" alt="aa"/></a>
          <img alt="bb"/>
          <img src="2.jpg"/>
        </root>


    ```scala
    import scala.xml._
    val root = XML.loadFile("/home/code/scala/16.xml")
    (root \\ "img").foreach { case n if (n.attribute("alt") == None) => print(n); case _ => }
    ```

5. 
沿用上题的16.xml

    ```scala
    import scala.xml._
    val root = XML.loadFile("/home/code/scala/16.xml")
    root \\ "img" \\ "@src" foreach {println _}
    ```

6. 

    ```scala
    import scala.xml._
    val root = XML.loadFile("/home/code/scala/16.xml")
    root \\ "a" foreach {a => println(a.text + "  " +a.attributes.get("href").getOrElse(""))}
    ```

7. 

    ```scala
    def makeXML(map: Map[String, String]) = {
      <dl>{map.map(p => <dt>{p._1}</dt><dd>{p._2}</dd>)}</dl>
    }
    ```

8. 

    ```scala
    import xml.Elem
    def makeMap(dl: Elem) = {
      (dl \ "dt" zip dl \ "dd").map(a => a._1.text -> a._2.text).toMap
    }
    ```

9. 
同10

10. 

测试用文件page.xml

        <!DOCTYPE html
        PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
        <html>
            <head>
                <title>simple document</title>
            </head>
            <body>
                <p>a simple paragraph
                    <img alt="aa"></img>
                    <img src="1.jpg"></img>
                </p>
                <img/>
            </body>
            <![CDATA[system.out.println()]]>
            <!-- comments here -->
        </html>

  代码如下

    ```scala
    import xml._
    import xml.transform._
    import xml.parsing.ConstructingParser
    import xml.dtd.DocType
    import java.io.File

    //val a = xml.XML.loadFile("/tmp/page.xml")
    //save DTD and CDATA
    val doc = ConstructingParser.fromFile(new File("/tmp/page.xml"),
      true).document
    val a = doc.docElem

    val imgRule = new RewriteRule {
      override def transform(n:Node) = n match {
        case i @ <img>{_*}</img> if(i.attribute("alt") == None) => 
        i.asInstanceOf[Elem] % Attribute(null, "alt", "TODO", Null)
        case _ => n
      }
    }

    val result = new RuleTransformer(imgRule).transform(a)
    XML.save("/tmp/page1.xml", result(0), "UTF-8", false,
      DocType("html", doc.dtd.externalID, Nil))
    ```

----
<div align="right">use Scala 2.11.1</div>
