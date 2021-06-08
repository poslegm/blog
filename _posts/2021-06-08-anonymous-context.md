---
layout: post
title: "Анонимные контекстные параметры"
date: 2021-06-08 00:00:00
description: "Удобство анонимных контекстных параметров в Scala 3"
---

Мелочь, а приятно: в третьей скале можно не придумывать имена для контекстных
параметров.

{% highlight scala %}
def example(fut: Future[Int])(using ExecutionContext): Future[String] =
  fut.map(_.toString)
{% endhighlight %}

вместо

{% highlight scala %}
def example(fut: Future[Int])(implicit ec: ExecutionContext): Future[String] =
  fut.map(_.toString)
{% endhighlight %}

из второй скалы.

Контекстные параметры очень часто нужны просто чтобы прокинуть их дальше или
задать ограничение на тип, поэтому в их именовании нет смысла. Так что с
анонимными параметрами и когнитивная нагрузка на придумывание имени пропадает,
и пространство имён не засоряется заведомо не используемыми значениями.

Подробнее о контекстных абстракциях хорошо написано в
[документации](https://dotty.epfl.ch/docs/reference/contextual/motivation.html).
