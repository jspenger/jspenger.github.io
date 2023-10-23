---
layout: post
title: "Scala Tricks"
date: 2023-01-15
tags: Scala 3
lastupdatedon: 2023-01-15
---

<!-- Select and load style sheet: https://highlightjs.org/static/demo/ https://cdnjs.com/libraries/highlight.js -->
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/10.7.2/styles/stackoverflow-light.min.css" integrity="sha512-cG1IdFxqipi3gqLmksLtuk13C+hBa57a6zpWxMeoY3Q9O6ooFxq50DayCdm0QrDgZjMUn23z/0PMZlgft7Yp5Q==" crossorigin="anonymous" />

<!-- Load js file: -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/highlight.js/10.7.1/highlight.min.js" integrity="sha512-d00ajEME7cZhepRqSIVsQVGDJBdZlfHyQLNC6tZXYKTG7iwcF8nhlFuppanz8hYgXr8VvlfKh4gLC25ud3c90A==" crossorigin="anonymous"></script>
<script>hljs.highlightAll();</script>

# Scala Tricks 
This is a collection of Scala tricks that I have found useful. Some of these are Scala 3 specific.

### Multiple type parameter lists for a single function
I wanted to split my type parameter list in two, so that one would be inferred, and the other provided by the user. This was especially important for this case as the inferred one had a very complicated type. I found the answer somewhere online, I don't remember the exact link at the time, but the approach was attributed to Miles Sabin [here](https://github.com/scala/bug/issues/4719).

```scala
trait Foo[A]:
  def apply[B](a: A, b: B): (A, B) = (a, b)
  def apply[B, C](a: A, b: B, c: C): (A, B, C) = (a, b, c)

new Foo[Int] {}.apply[String](1, "2") // : Tuple2[Int, String]
new Foo[Int] {}.apply[String, Int](1, "2", 3) // : Tuple3[Int, String, Int]

def foo[A] = new Foo[A] {}

foo(1, "2") // : Tuple2[Int, String]
foo(1, "2", 3) // : Tuple3[Int, String, Int]
foo[Int][String, Int](1, "2", 3) // : Tuple3[Int, String, Int]

// Alternative:
object Foo:
  def apply[A] = new Foo[A] {}

Foo(1, "2") // : Tuple2[Int, String]
Foo(1, "2", 3) // : Tuple3[Int, String, Int]
Foo[Int][String, Int](1, "2", 3) // : Tuple3[Int, String, Int]
```

### Match Types and Tuples
The new Scala 3 tuple types are both fun to use, and I have found them very useful. The tuple type is constructed much like a cons-list, and we can deconstruct it similarly using match types.

The Scala standard library already provides very useful match types for working with tuples. Here I will paste the `Head` and the `Map` match types. They can be accessed by writing `Tuple.Head` and `Tuple.Map` respectively. I simply refer to the standard library, the examples there are very useful for seeing match types in action.

```scala
/** Type of the head of a tuple */
  type Head[X <: NonEmptyTuple] = X match {
    case x *: _ => x
  }

/** Converts a tuple `(T1, ..., Tn)` to `(F[T1], ..., F[Tn])` */
  type Map[Tup <: Tuple, F[_ <: Union[Tup]]] <: Tuple = Tup match {
    case EmptyTuple => EmptyTuple
    case h *: t => F[h] *: Map[t, F]
  }
```

### Type Predicates and Implicit Evidence
Another thing that I have been playing with is using match types and implicit evidence for constructing something which I call type predicates (I am sure there is a better name for this). I have been using this as an internal API, trying to restrict certain type parameters to certain properties, for example to ensure that a type parameter is not Nothing. I am not 100% sure if this works, but so far have not had any issues with it. 

```scala
object Predicates:
  /** If type T is not Nothing */
  type NotNothing[T] = true =:= T match
      case Nothing => false
      case _       => true

  object NotNothing:
    /** implcitNotFound message */
    inline val MSG = "Type T is ${T}, it is required to NOT be Nothing"
```

The example usage is to add it as a guard on a function, to ensure that the type parameter is not Nothing.

```scala
import scala.annotation.implicitNotFound
import Predicates.*

def foo[T]()(using @implicitNotFound(NotNothing.MSG) ev: NotNothing[T]): Unit =
  print("foo")

foo[Int]()

// compiler error: Type T is Any, it is required to NOT be Nothing
// foo() 

// compiler error: Type T is Nothing, it is required to NOT be Nothing
// foo[Nothing]() 
```

### Type-Safe Config
The idea of Type Predicates is taken to the extreme with a type-safe config that I built (mostly for fun, not much profit), using Heterogeneous Maps.
This type of pattern is described well in other sources, but I forgot the links, it should suffice to google for "Heterogeneous Maps in Scala 3".
Check out [Config.scala](https://github.com/portals-project/portals/blob/main/portals-libraries/src/main/scala/portals/libraries/actor/Config.scala).