---
layout: post
title: "Тайптеги для зависимостей в ZIO"
date: 2021-10-19 00:00:00
description: "Как использовать тайптеги для автовайринга ZLayer"
---

В DI-фреймворке под названием ZIO можно автовайрить зависимости с помощью
расширения [zio-magic](https://github.com/kitlangton/zio-magic), который ещё и
встроят в версию 2.0. Код выглядит примерно так:

{% highlight scala %}
ZLayer.fromMagic[Env](
  UsersRepository.live,
  ClientsRepository.live,
  RegistrationService.live
)
{% endhighlight %}

А дальше макрос на этапе компиляции выстроит граф зависимостей по входящим
типам слоёв. Но частенько в приложении бывают классы, принимающие зависимости
одного и того же типа. Допустим, grpc-клиенты к микросервисам `BillingClient` и
`NotificatorClient` принимают в зависимости `GrpcConfig`. Содержимое этих
конфигов очевидно разное, но на уровне типов этого не видно, поэтому нельзя
понять, какой экземляр конфига к кому относится. Можно решить это ручным
вайрингом:

{% highlight scala %}
ZLayer.fromMagic[Env](
  Grpc.live,
  Config.loadLayer[GrpcConfig]("billing") ++ ZLayer.environment[Has[Grpc]] >>> BillingClient.live,
  Config.loadLayer[GrpcConfig]("notificator") ++ ZLayer.environment[Has[Grpc]] >>> NotificatorClient.live
)
{% endhighlight %}

Но мне это не нравится, потому что приходится внедрять куски ручного построения
графа зависимостей в плоский список слоёв.

Так как вайринг происходит во время компиляции, различать зависимости нужно на
уровне типов. Сразу пришла идея использовать
[тайп-теги](https://medium.com/iterators/to-tag-a-type-88dc344bb66c): пометить
слой зависимости каким-то тегом, и такой же тип затребовать в слое-получателе.
Собственно, после некоторого пердолинга с компилятором было написано такое вот
поделие:
[https://gist.github.com/poslegm/f252994a15453457e64d6498249928f3](https://gist.github.com/poslegm/f252994a15453457e64d6498249928f3). 

И с ним можно писать уже так:

{% highlight scala %}
ZLayer.fromMagic[Env](
  Grpc.live,
  Config.loadLayer[GrpcConfig]("billing").tagged[BillingClient.Service],
  Config.loadLayer[GrpcConfig]("notificator").tagged[NotificatorClient.Service],
  BillingClient.live.requireTagged[GrpcConfig, BillingClient.Service, Has[Grpc]],
  NotificatorClient.live.requireTagged[GrpcConfig, NotificatorClient.Service, Has[Grpc]]
)
{% endhighlight %}

К сожалению не получилось придумать реализацию `requireTagged`, в которой не
надо явно указывать компилятору тип "остатка" требуемых слоёв, кроме
протегированного. Зато теперь все зависимости укладываются в один плоский
список, а zio-magic ругнётся, если не будет протегированной зависимости к
какому-то слою.
