---
layout: post
title: "О перфомансе"
date: 2021-10-05 00:00:00
description: "О производительности языков программирования по мотивам Ruby Russia"
---

Посмотрел на Ruby Russia кейноут создателя языка. Там он всерёз говорит об
оптимизации языка под микробенчмарки, потому что программисты слишком серьёзно
к ним относятся, и выбирают языки, которые обгоняют Ruby. То есть буквально не
из-за того, что ускорение рантайма принесёт какую-то ценность, а ради
увеличения привлекательности языка. Дальше рассказывал про целый роадмап
повышения перфоманса Ruby: многоуровневый JIT, гранты контрибуторам за
микрооптимизации. И всё это под соусом того, что "медленность" вредит
маркетингу.

И вот не понимаю: программисты действительно столько внимания уделяют
синтетическим бенчмаркам? Почему?

Конечно, есть критичные к перфомансу области разработки, но они уже заняты
специфичными языками: Rust, C++. На Ruby пишутся в основном веб-бэкенды, а в
них, чтобы упереться в производительность *языка*, надо особо постараться. Если
в сервисе что-то тормозит, то почти всегда накосячил программист, а не рантайм.
На своей практике припоминаю только два случая, когда тормоза на высоких
нагрузках можно было с натяжкой списать на проблемы платформы. Оба связаны с
(де-)сериализацией, что характерно :). Но и они решились не переписыванием со
скалы на раст, а просто заменой библиотеки и небольшими оптимизациями кода. 

А что до синтетических бенчмарков: в самом популярном
[сравнении](https://www.techempower.com/benchmarks/) HTTP серверов akka-http
стабильно болтается около 190-го места. При этом в проде работает нормально и
кушать не просит. Если что-то и тормозит, то в коде приложения, а не
фреймворка. Современное состояние индустрии таково, что фреймворк со дна
бенчмарков может держать внушительную нагрузку! И это не говоря о дешёвом
железе. 

Поэтому в моих глазах топ причин плохого перфоманса выглядит так:

1. баг в коде;
2. ошибка конфигурации; 
3. ошибка дизайна системы;
4. ... программист обосрался где-то ещё ...
5. медленный рантайм языка.

Или в руби всё настолько плохо с производительностью, что даже скала по
сравнению с ней летает? Или программистам комфортно списывать свои оплошности
на "медленный язык"?
