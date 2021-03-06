## Анализируем время ответа собеседника

С появлением мессенджеров коммуникация перешла на новый уровень -- возможность мгновенного доступа к собеседнику воспринимается теперь как должное.

Но замечали ли вы, как на ваши ощущения от общения влияет скорость его ответа? Какое время ответа вообще считается приемлемым?

Можем ли мы сказать, что проявляем неуважение, когда отвечаем на следующий день? Через неделю? Через месяц?

В этой статьей мы не будем отвечать на эти вопросы. Зато без каких-либо глобальных выводов проведем небольшое исследование одного параметра -- время ответа собеседником на наши сообщения.

# Достаем сырые данные

Для исследования в нашем случае лучше всего подойдет Telegram. Прежде всего, потому что у него есть удобный api для Python.

Будем использовать библиотеку telethon (вот ее документация https://docs.telethon.dev/en/latest/).

Код загрузки истории переписки весьма лаконичен:
```
    username = '<user>'

    user = await client.get_entity(username)

    timestamps_history = []

    offset = 0
    has_messages = True
    while has_messages:
        history = await client(GetHistoryRequest(
            peer=user,
            limit=200,
            offset_date=None,
            offset_id=0,
            max_id=0,
            min_id=0,
            add_offset=offset,
            hash=0))

        has_messages = False
        for message in history.messages:
            has_messages = True
            timestamps_history.append((message.date, message.out, message.message))

        offset += len(history.messages)
        if offset % 1000 == 0:
            print(offset)
```

Полностью скрипт загрузки и обработки сообщений можно увидеть здесь https://github.com/evgenstf/response_timings_research.



Для его выполнения на своей переписке, вам при первом запуске нужно авторизоваться по номеру телефона и коду безопасности.

Telethon возвращает сообщения в удобном формате со всеми необходимыми параметрами: нам нужно время отправки, отправитель и собственно сам текст.

# Извлекаем времена ответа

Есть несколько вариантов величин, которые можно исследовать. Например, можно разделить диалог на *реплики* -- последовательные сообщения от одного отправителя. Тогда в качестве исследуемых времен можно использовать задержки между нашими репликами и собеседника.

Однако более показательными и интересными будут времена ответов на явные вопросы -- сообщения содержащие '?' на конце.

# Строим распределение

Итак, у нас есть измеренные времена ответов собеседника на наши вопросы. Что с этим делать дальше? Самое простое и первое что приходит на ум -- посчитать медиану и среднее значение.

```
friend: her median: 73 my median: 38
friend: her mean: 5823.03 my mean: 3841.03

mom: her median: 15 my median: 21
mom: her mean: 352.32 my mean: 77.25

colleague: her median: 20.0 my median: 15
colleague: her mean: 815.08 my mean: 204.84

classmate: his median: 63 my median: 18
classmate: his mean: 2656.09 my mean: 554.58

ex: her median: 35 my median: 18.0
ex: her mean: 586.59 my mean: 999.27
```

Можно видеть, что для разных людей мое личное значение времени реакции различается.


Но, поскольку хочется что-то более, чем два числа, мы построим распределение этого значения:

![](https://habrastorage.org/webt/gt/-e/mo/gt-emohiidsx1hyflmjzvgnu-9u.png)
Из него видна проблема в данных -- на больших временах значения довольно сильно разбросаны. Это можно исправить, Попробуем сделать шкалу времени не линейной, а логарифмической. Посколько и в жизни значимость времени ответа логарифмически уменьшается (довольно существенно, ответил ли собеседник через 5 минут или через 10, однако через день эта разница уже не столь значительна).

![](https://habrastorage.org/webt/oe/mt/rp/oemtrpo9yczsyuv7zhj94deffgi.png)


Ну и в конце для каждой персоны можем добавить аналогичный анализ для времен наших ответов. В целом это может показывать насколько мы более заинтересованы в общении с собеседником, по сравнению с ним. Но куда более точно можно быть уверенным, что заинтересованность в общении прослеживается при сравнении нашей реакции на разных собеседниках.

![](https://habrastorage.org/webt/ry/lc/35/rylc35fhbxt2achjolnqeswvrlc.png)

Можно видеть, что мы отвечаем на вопросы чаще: распределение ответов смещено к 7ми секундам, против 45и у собеседника.

# Сравнение с разными людьми

Интересно сравнить, как меняется распределение в зависимости от отношений с человеком.

Ниже приведены несколько примеров:

## Коллега по работе
![](https://habrastorage.org/webt/gm/qh/yy/gmqhyyj6k5irzlextcnz5ob60qk.png)
## Девушка
![](https://habrastorage.org/webt/vu/dx/f5/vudxf5bj2vvwmbwx8baiisb_l08.png)
## Друг
![](https://habrastorage.org/webt/5o/d-/5l/5od-5lmpm2n2bswudutgftzesia.png)



Как и обещали, никаких глобальных выводов не будет. Общайтесь так, как вам комфортно, не оглядываясь на этикет.
