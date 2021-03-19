---
layout: post
title: "Плохая производительность fold"
date: 2020-03-26 00:00:00
description: "Как Travis Brown Validated оптимизировал"
---

Наткнулся на старый [тикет](https://github.com/typelevel/cats/issues/1951), в котором Travis Brown померял производительность вызова fold по сравнению с обычным паттерн-матчингом. Замеры он делал для `cats.data.Validated`, но результаты актуальны и для аналогичных контейнеров (`Either`, `Option`). А результаты неутешительные — `fold` медленнее заинлайненного паттерн-матчинга в четыре раза.

`fold` для этих типов реализован тривиально и фактически принимает две функции, чтобы засунуть их в паттерн-матчинг:

{% highlight scala %}
sealed abstract class Validated[+E, +A] {
  // ...
  def fold[B](fe: E => B, fa: A => B): B = this match {
    case Invalid(e) => fe(e)
    case Valid(a) => fa(a)
  }
  // ...
}

Validated   // <- медленно
  .valid(42)
  .fold(e => println(s"error $e"), res => println(s"result $res"))

Validated.valid(42) match {   // <- быстро
  case Invalid(e) => println(s"error $e")
  case Valid(res) => println(s"result $res")
}
{% endhighlight %}

Просадка производительности связана с тем, как в jvm реализованы лямбды: для каждой лямбды аллоцируется объект в куче. Соотвественно `fold` только и делает, что аллоцирует лямбды для своих аргументов.

Что с этим делать? Да ничего. Если у вас есть горячий код с фолдами, то вы, скорее всего, уже давно всё заинлайнили. Если код холодный, то оверхед вы и не заметите.
