---
layout: post
title:  "Шпаргалка по командам кафки"
date:   2018-03-31 00:00:00
---

За полгода работы с кафкой накопился набор часто используемых команд. А из-за того, что CLI интерфейс кафки совершенно неудобный, держать команды в голове невозможно, и приходится писать очередную шпаргалку. 

Больше всего раздражает разделение команд по нескольким утилитам. Получается, что для выполнения одного действия необходимо знать не только аргументы командной строки, но и название конкретной утилиты. А получилось так потому что админские команды отправляются в кафку через **однострочные баш-скрипты, запускающие Scala-классы**.

```sh
exec $(dirname $0)/kafka-run-class kafka.admin.ConfigCommand "$@"
```

Поэтому приходится вести заметку с нужными командами

####  Посмотреть оффсеты всех топиков

```sh
kafka-consumer-groups --bootstrap-server kafka.dev.big-company.com:9092 --describe --group group.dev
```

Выводит оффсеты и лаги группы `group.dev` по всем топикам. Каждая партиция отдельно. 

#### Сброс оффсетов

```sh
kafka-consumer-groups --bootstrap-server kafka.dev.big-company.com:9092 --reset-offsets --to-offset 340000 --group group.dev --topic actions --execute
```

Выставляет оффсет топика `actions` на указанное значение, если это возможно. Консьюмеры должны быть выключены в момент исполнения команды.

```sh
kafka-consumer-groups --bootstrap-server kafka.dev.big-company.com:9092 --reset-offsets --to-latest --group group.dev --topic actions --execute
```

Выставляет оффсет топика `actions` на самое раннее возможное значение. Консьюмеры должны быть выключены. 

Для предыдущих двух команд можно указать `--all-topics` вместо названия топика.

#### Удаление и создание топиков

```sh
kafka-topics --zookeeper zk.dev.big-company.com --create --replication-factor 2 --partitions 2 --topic actions
```

Создаёт топик с указанным количеством реплик и партиций.

```sh
kafka-topics --zookeeper zk.dev.big-company.com --delete --topic actions
```

Удаление топика.

#### Просмотр информации о топиках

```sh
kafka-topics --zookeeper zk.dev.big-company.com --list
```

Выводит список всех топиков.

```sh
kafka-topics --zookeeper zk.dev.big-company.com --describe
```

Выводит всю информацию по топикам: кастомные конфиги, количество реплик и партиций, лидирующую партицию.

```sh
kafka-configs --zookeeper zk.dev.big-company.com --describe --entity-type topics --entity-name actions
```

Выводит кастомные конфиги топика, то есть параметры, которые перегружают основной конфигурационный файл кафки.

#### Конфигурация топиков

```sh
kafka-topics.sh --zookeeper zk.dev.big-company.com --alter --topic actions --config segment.bytes=104857600
```

Выставляет значение `segment.bytes` для топика `actions`.

```sh
kafka-topics.sh --zookeeper zk.dev.big-company.com --alter --topic actions --delete-config segment.bytes
```

Удаляет кастомный параметр. Для этого топика будет использоваться параметр из общего конфига.

#### Очистка сообщений топика

Отдельной команды для этого нет, поэтому надо хитрить с таймаутом, после которого сообщения удаляются с жёсткого диска.

```sh
kafka-topics.sh --zookeeper zk.dev.big-company.com --alter --topic actions --config retention.ms=1000
```

Ждём 1000 миллисекунд...

```sh
kafka-topics.sh --zookeeper zk.dev.big-company.com --alter --topic mytopic --delete-config retention.ms
```