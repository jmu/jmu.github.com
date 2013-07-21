---
layout: post
title: "Scala in Impatient 习题解答5"
description: "快学Scala习题答案"
category: Scala
tags: [Scala, 快学Scala, Scala for the Impatient]
---
{% include JB/setup %}

1. 

        class Counter {
            private var value = 0

            def increment() { if (value < Int.MaxValue) value += 1 }
            def current = value
        }

2. 

        class BankAccount {
            private var b = 0

            def balance = b 

            def deposit(num: Int) = {
                if (num > 0) {
                    b = b + num
                }
                b
            }

            def withdraw(num: Int) = {
                if (num > 0) {
                    b = b - num
                }
                b
            }
        }

3. 

        class Time(val hours: Int, val minutes: Int) {

            def before(other: Time): Boolean = {
                hours + minutes < other.hours + other.minutes
            }
        }

        scala> val a = new Time(3,21)
        a: Time = Time@13a09b0

        scala> val b = new Time(23,39)
        b: Time = Time@1da6308

        scala> a.before(b)
        res23: Boolean = true

        scala> b.before(a)
        res24: Boolean = false

4. 

        class Time(val hours: Int, val minutes: Int) {
            private val m = hours * 60 + minutes

            def before(other: Time): Boolean = {
                m < other.m
            }
        }

5. 

        class Student {
            var name: String = ""
            var id: Long = 0
        }

        $ javap -private Student.class 
        Compiled from "5.scala"
        public class Student implements scala.ScalaObject {
          private java.lang.String name;
          private long id;
          public java.lang.String name();
          public void name_$eq(java.lang.String);
          public long id();
          public void id_$eq(long);
          public Student();
        }

  使用`@BeanProperty`增加兼容JavaBean的get和set方法

        import scala.reflect.BeanProperty

        class Student {
            @BeanProperty
            var name: String = _
            @BeanProperty
            var id: Long = 0
        }

6. 

        class Person(var age: Int) {
            if (age < 0) age = 0
        }

7. `name`应该设定为`val`以防止被修改

        class Person(val name: String) {
            private val nameArray = name.split(" ")

            def firstName = nameArray(0)
            def lastName = nameArray(1)
        }

8. 

        class Car(val manufacturer: String, val model: String,
            val year:Int = -1, var number:String = "") {
        }

9. 感受下Java实现的

        public class Car { 
            private final String manufacturer;
            private final String model;
            private final int year;
            private String number;

            public Car(String manufacturer, String model) {
                this(manufacturer, model, -1, "");
            }

            public Car(String manufacturer, String model, int year) {
                this(manufacturer, model, year, "");
            }

            public Car(String manufacturer, String model, int year, String number) {
                this.manufacturer = manufacturer;
                this.model = model;
                this.year = year;
                this.number = number;
            }

            public String getManufacturer() {
                return this.manufacturer;
            }

            public String getModel() {
                return this.model;
            }

            public int getYear() {
                return this.year;
            }

            public String getNumber() {
                return this.number;
            }

            public void setNumber() {
                this.number = number;
            }
        }

10. 显然还是用`primary constructor`更简洁

        class Employee {
          private var n: String = "John Q. Public"
          var salary: Double = 0.0

          def this(name: String, salary: Double) {
            this()
            n = name
            this.salary = salary
          }

          def name = n
        }

----
<div align="right">use Scala 2.9.1</div>
