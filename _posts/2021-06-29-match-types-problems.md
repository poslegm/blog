---
layout: post
title: "Что не понравилось в Match Types"
date: 2021-06-29 00:00:00
description: "Мои проблемы с Match Types в Scala 3"
---

Поигрался с [match
types](https://dotty.epfl.ch/docs/reference/new-types/match-types.html) в Scala 3.
Сильно хотел использовать их для определения зависимости возвращаемого типа
функции от типа аргумента, но не прокатило. Не понравилось, что матчинг при
использовании типа происходит в рантайме, поэтому можно словить исключение
`MatchError`:

{% highlight scala %}
type Invert[T] = T match
  case String => Int
  case Int => String

def invert[T](t: T): Invert[T] =
  t match
    case s: String => s.length
    case i: Int => i.toString

val test = invert(false) // упадёт в рантайме
{% endhighlight %}

[Полный пример в
Scastie](https://scastie.scala-lang.org/A5o7eCFwSLeZZc70kBeuLQ). Для переменной
`test` компилятор выводит тип `Invert[Boolean]`, но не проверяет, есть ли такая
ветка в определении типа. В итоге код компилируется, но падает при запуске.

Второе следствие матчинга в рантайме — потеря ленивости аргументов:

{% highlight scala %}
type Eval[T] = T match
  case Any => "do nothing"

def eval[T](t: => T): Eval[T] =
  t match
    case _: Any => "do nothing"

val a = eval {
  println("oooops")
  42
}
{% endhighlight %}

[Полный пример в
Scastie](https://scastie.scala-lang.org/KokFwColQOCNpVD8ygCIvw).

Пока не понимаю, является ли матчинг в рантайме следствием фундаментальных
ограничений языка. Но вообще хотелось бы иметь возможность без костылей
сделать зависимую типизацию для своей функции, просто разматчив тип
аргумента.

Появление
[summonFrom](https://dotty.epfl.ch/docs/reference/metaprogramming/compiletime-ops.html#summoning-implicits-selectively)
и [transparent
inline](https://dotty.epfl.ch/docs/reference/metaprogramming/inline.html#transparent-inline-methods)
позволяет накостылить нечто подобное чуть менее вербозно, чем в Scala 2, но всё
равно не интуитивно.

Пример на Scala 2:
[https://github.com/zio/zio/issues/5241](https://github.com/zio/zio/issues/5241)

Он же на Scala 3:
[https://scastie.scala-lang.org/EwfBu7rcTwi6UQ11VYMwEQ](https://scastie.scala-lang.org/EwfBu7rcTwi6UQ11VYMwEQ)

Несмотря на то, что эта штука _работает_, выглядит она не как нативная языковая
конструкция, а как насилие над компилятором.
