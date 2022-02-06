---
layout: post
title: "Applicative for"
date: 2022-02-06 00:00:00
description: "for-comprehension для параллельных вычислений"
---

Kit Langton, автор [макроса для автовайринга
ZLayer](https://github.com/kitlangton/zio-magic), отметился в любопытном
[пропозале](https://contributors.scala-lang.org/t/for-syntax-for-parallel-computations-afor-applicative-for-comprehension/4474/1)
на форуме разработчиков компилятора Scala. В пропозале обсуждается
синтаксический сахар для аппликативной композиции по аналогии с
`for-comprehension` для монадок. Если сильно упрощать смысл аппликативных
вычислений до наиболее частого практического применения -- это вычисление
нескольких эффектов параллельно и сбор результатов в одном месте (академики, не
ругайтесь).

Сейчас в популярных библиотеках параллельные вычисления описываются
вспомогательными функциями, и это не особо удобно в реальности, когда собрать
надо с десяток значений:

{% highlight scala %}
ZIO.mapParN(fetchName, fetchAge)((name, age) => User(name, age))
{% endhighlight %}

А программисты хотят синтаксического сахарка, как если бы вычисления были
последовательными:

{% highlight scala %}
afor {
  name <- fetchName
  age <- fetchAge
} yield User(name, age)
{% endhighlight %}

В предложенном варианте меньше бросается в глаза, что вычисления побегут в
параллель, но читается он гораздо лучше. В Haskell такое [расширение
компилятора](https://gitlab.haskell.org/ghc/ghc/-/wikis/applicative-do) уже
реализовано и выглядит довольно симпатично.

Так вот, Kit написал [работающий PoC
макроса](https://github.com/kitlangton/parallel-for/), который даёт такой
синтаксис без доработок компилятора. Пока только для ZIO. 

{% highlight scala %}
par {
  for {
    name <- fetchName
    age <- fetchAge
  } yield User(name, age)
}
{% endhighlight %}

Макрос строит граф вычислений из обёрнутого `for-comprehension`, топологически
его сортирует и параллелит всё, что можно. Утилита уже сейчас выглядит
привлекательно, и может быть её втащат в ZIO, как это уже было с zio-magic.

https://github.com/kitlangton/parallel-for
