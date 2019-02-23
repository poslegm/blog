---
layout: post
title:  "Микрооптимизация scala-phash"
date:   2019-02-21 00:00:00
description: "Оптимизация Scala кода с помощью замены map и foreach на циклы while"
---

Прочитав статью Li Haoyi про [микрооптимизации](http://www.lihaoyi.com/post/MicrooptimizingyourScalacode.html) Scala-кода, решил проверить эффект одной из них самостоятельно. В качестве подопытного кролика выбрал свою библиотеку сравнения изображений [scala-phash](https://github.com/poslegm/scala-phash). 

Запускаю юнит-тесты, подключаюсь к процессу через VisualVM и вижу в топе процессорного времени очень интересные функции `map` и `foreach`. 

![VisualVM](/assets/images/scala-while/vm.png)

Self Time отображает время, которое тратится на исполнение самого метода без учёта тех функций, которые он вызывает изнутри. То есть куча времени уходит именно на итерирование по массивам, а не на действия, которые осуществляются с их элементами.

Об этой проблеме писал и Li Haoyi в своей статье: проходы по массивам при помощи `map` и `foreach` работают гораздо менее эффективно, чем простой цикл `while`. Вероятно, это связано со спецификой исполнения Scala кода на jvm, в частности, боксингом ― заворачиванием примитивного типа в объект (например, `int -> java.lang.Integer`). `foreach` принимает обобщённое лямда-выражение, параметр которого компилятору приходится боксить.

Кроме прямых вызовов упомянутых методов, в моём коде есть обход массивов через `for`. Это тоже плохая идея, потому что при компиляции он развернётся в тот же самый `foreach`:

{% highlight scala %}
// Первая строка при компиляции развернётся во что-то, очень похожее на вторую 
for (i <- Array(1, 2, 3, 4, 5)) { println(i) }
Array(1, 2, 3, 4, 5).foreach(i => println(i))
{% endhighlight %}

В то же время `while` ― это не метод, принимающий лямбду, а самостоятельная языковая конструкция. Поэтому циклы лишены проблем с боксингом.

Если отбросить детали, то моя библиотека выполняет квадратичные алгоритмы, поэлементно перебирающие большие числовые массивы. Поэтому удалось получить хороший прирост производительности, просто [переписав](https://github.com/poslegm/scala-phash/commit/deb5b01b923b9077bf98ab36734af760f083e26e) обход трёх массивов на `while`, не затрагивая алгоритмическую сложность программы.

Время на выполнение тестов до переписывания:
![sbt test before](/assets/images/scala-while/before.png)
И после:
![sbt test after](/assets/images/scala-while/after.png)

Ускорение почти на 40% без каких-либо умственных усилий!

Для верности написал ещё простой бенчмарк с хэшированием изображения 500x500px новой и старой версиями библиотеки:

{% highlight scala %}
import org.scalameter._
import scalaphash._

import java.awt.image.BufferedImage
import java.io.File
import javax.imageio.ImageIO

object Hello extends App {
  val img = ImageIO.read(new File("src/main/resources/images/small.jpg"))

  bench("radial", img)(PHash.unsafeRadialHash(_))

  bench("marr", img)(PHash.unsafeMarrHash(_))

  bench("DCT", img)(PHash.unsafeDctHash(_))

  private def bench[Hash](name: String, image: BufferedImage)(f: BufferedImage => Hash): Unit = {
    val time = config(
      Key.exec.benchRuns -> 20
    ) withWarmer {
      new Warmer.Default
    } withMeasurer {
      new Measurer.IgnoringGC
    } measure {
      f(image)
    }
    println(s"$name time: $time")
  }
}
{% endhighlight %}

Результаты замеров очень приятные:

```
1.2.1

radial time: 82.82101674999998 ms
marr time: 595.6078000499999 ms
DCT time: 96.3602538 ms

1.2.0

radial time: 164.72333365000003 ms
marr time: 1253.7434706 ms
DCT time: 292.0225023500001 ms
```

Несмотря на дешевизну этой оптимизации, она имеет смысл только для очень небольшого подмножества программ. Эффект будет заметен только при итерациях по большим массивам и **только** в горячем коде. В большинстве случаев бутылочное горлышко расположено далеко не в обходе коллекций.
