---
layout: post
title: "Monadic Reflection"
date: 2021-06-05 00:00:00
description: "Об экспериментальной библиотеке из EPFL для монадической композиции"
---

В EPFL не только релизят третью скалу, но и продолжают шатать монадки. Вслед за
заходами в [async/await](https://github.com/scala/scala-async) и [effects as
abilities](http://dotty.epfl.ch/docs/reference/experimental/canthrow.html#from-effects-to-abilities)
появилась экспериментальная библиотека [Monadic
Reflection](https://github.com/lampepfl/monadic-reflection) (от создателя не
менее экспериментального
[scala-effect](https://github.com/b-studios/scala-effekt)).

И это очередная попытка избавиться от необходимости писать flatMap или
for-comprehension при работе с монадками. То есть вместо 

{% highlight scala %}
for
  _ <- f()
  _ <- g()
yield ()
{% endhighlight %}

предлагается написать просто

{% highlight scala %}
f()
g()
{% endhighlight %}

а дальше оно *само скомпозится*.

Библиотека работает только с Project Loom (если до сех пор не знаете о Loom,
бегом
[читать](https://blogs.oracle.com/javamagazine/going-inside-javas-project-loom-and-virtual-threads)
/ [смотреть](https://www.youtube.com/watch?v=-CPWbB-Pn14)). Каждый шаг
монадической композиции
[запускается](https://github.com/lampepfl/monadic-reflection/blob/main/core/src/main/scala/monadic/Monadic.scala#L48-L54)
на лумовом `continuation`. Вот [примерчик с
ZIO](https://github.com/lampepfl/monadic-reflection/blob/main/zio/src/main/scala/monadic/examples.scala#L57-L66).

Кажется, что в EPFL понимают, что *монады — это паттерн императивного
программирования*, и всячески пытаются срезать синтаксические излишки, чтобы
избавиться от разницы между "обычным" императивным кодом и завёрнутым в
монадки.

Выглядит интересно, и возможно такие эксперименты приведут индустрию в светлое
будущее со ссылочной прозрачностью императивного кода без накладных расходов на
синтаксис. Но на мой взгляд написание флэтмапов не сильно снижает
продуктивность разработчика, поэтому непонятна выгода от их оптимизации. Те же
[abilities](http://dotty.epfl.ch/docs/reference/experimental/canthrow.html#from-effects-to-abilities)
выглядят интереснее.
