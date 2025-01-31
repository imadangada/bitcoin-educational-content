---
name: Payjoin
description: Что такое Payjoin в Bitcoin?
---
![Миниатюра Payjoin - стеганография](assets/cover.webp)

***ВНИМАНИЕ:** После ареста основателей кошелька Samourai и изъятия их серверов 24 апреля, функция Payjoins Stowaway в кошельке Samourai работает только путем ручного обмена PSBT между заинтересованными сторонами, при условии, что оба пользователя подключены к своему Dojo. Что касается Sparrow, Payjoins через BIP78 по-прежнему работают. Однако возможно, что эти инструменты будут перезапущены в ближайшие недели. Тем временем вы все еще можете прочитать эту статью, чтобы понять теоретические принципы работы payjoins.*

_Мы внимательно следим за развитием этого дела, а также за развитием связанных с ним инструментов. Будьте уверены, что мы обновим этот учебник, как только появится новая информация._

_Этот учебник предоставляется только для образовательных и информационных целей. Мы не поддерживаем и не поощряем использование этих инструментов в преступных целях. Каждый пользователь несет ответственность за соблюдение законов в своей юрисдикции._

---
## Понимание транзакций Payjoin в Bitcoin

Payjoin - это специфическая структура транзакции Bitcoin, которая улучшает конфиденциальность пользователя во время платежа за счет сотрудничества с получателем платежа.

В 2015 году [LaurentMT](https://twitter.com/LaurentMT) впервые упомянул этот метод как "стеганографические транзакции" в документе, доступном [здесь](https://gist.githubusercontent.com/LaurentMT/e758767ca4038ac40aaf/raw/c8125f6a3c3d0e90246dc96d3b603690ab6f1dcc/gistfile1.txt). Эта техника позже была принята кошельком Samourai, который стал первым клиентом, реализовавшим ее с помощью инструмента Stowaway в 2018 году. Концепция Payjoin также находится в [BIP79](https://github.com/bitcoin/bips/blob/master/bip-0079.mediawiki) и [BIP78](https://github.com/bitcoin/bips/blob/master/bip-0078.mediawiki). Существует несколько терминов для обозначения Payjoin:
- Payjoin
- Stowaway
- P2EP (Pay-to-End-Point)
- Стеганографическая транзакция

Уникальность Payjoin заключается в его способности генерировать транзакцию, которая на первый взгляд кажется обычной, но на самом деле является мини Coinjoin между двумя сторонами. Для достижения этого структура транзакции включает получателя платежа вместе с фактическим отправителем во входы. Получатель включает платеж самому себе в середине транзакции, что позволяет ему получить оплату.

Давайте рассмотрим конкретный пример: если вы покупаете багет за `4000 сатоши`, используя UTXO на `10,000 сатоши`, и выбираете Payjoin, ваш пекарь добавит UTXO на `15,000 сатоши`, принадлежащее ему, в качестве входа, который они полностью получат в качестве выхода, в дополнение к вашим `4000 сатоши`:
![Диаграмма транзакции Payjoin](assets/en/1.webp)
В этом примере пекарь вносит в качестве входа `15,000 сатоши` и получает `19,000 сатоши`, с разницей ровно в `4000 сатоши`, что и является ценой багета. С вашей стороны, вы входите с `10,000 сатоши` и заканчиваете с `6,000 сатоши` на выходе, что представляет собой баланс `-4000 сатоши`, что также является ценой багета. Для упрощения примера я намеренно опустил комиссии за майнинг в этой транзакции.

## Какова цель транзакции Payjoin?

Транзакция Payjoin преследует две цели, которые позволяют пользователям повысить конфиденциальность своего платежа.
Прежде всего, Payjoin направлен на введение в заблуждение внешнего наблюдателя, создавая ложный след в анализе блокчейна. Это становится возможным благодаря эвристике общего владения входами (CIOH). Обычно, когда транзакция в блокчейне имеет несколько входов, предполагается, что все эти входы, скорее всего, принадлежат одному и тому же субъекту или пользователю. Таким образом, когда аналитик рассматривает транзакцию Payjoin, он склонен полагать, что все входы исходят от одного и того же человека. Однако это восприятие неверно, поскольку получатель платежа также вносит входы наряду с фактическим плательщиком. Следовательно, анализ цепочки направляется к интерпретации, которая оказывается ложной.

Кроме того, Payjoin также позволяет ввести в заблуждение внешнего наблюдателя относительно фактической суммы совершенного платежа. Изучая структуру транзакции, аналитик может поверить, что платеж эквивалентен сумме одного из выходов. Однако на самом деле сумма платежа не соответствует ни одному из выходов. Это на самом деле разница между выходным UTXO получателя и входным UTXO получателя. В этом смысле транзакция Payjoin попадает в область стеганографии. Она позволяет скрыть фактическую сумму транзакции внутри фиктивной транзакции, которая выступает в качестве ложного следа.

> Стеганография - это техника сокрытия информации внутри других данных или объектов таким образом, что наличие скрытой информации не воспринимается. Например, секретное сообщение может быть скрыто внутри точки в тексте, который не имеет к этому никакого отношения, делая его незаметным для невооруженного глаза (это техника микроточки). В отличие от шифрования, которое делает информацию непонятной без ключа дешифрования, стеганография не изменяет информацию. Она остается отображенной на виду. Ее цель скорее в том, чтобы скрыть существование секретного сообщения, тогда как шифрование явно указывает на наличие скрытой информации, хотя и недоступной без ключа.

Давайте вернемся к нашему примеру транзакции Payjoin за покупку багета.
![Схема транзакции Payjoin снаружи](assets/en/2.webp)
Видя эту транзакцию в блокчейне, внешний наблюдатель, следующий обычным эвристикам анализа цепочки, интерпретировал бы ее следующим образом: "*Алиса объединила 2 UTXO в качестве входов транзакции, чтобы заплатить `19,000 сатоши` Бобу*."
![Неверная интерпретация транзакции Payjoin снаружи](assets/en/3.webp)
Эта интерпретация очевидно неверна, потому что, как вы уже знаете, два входных UTXO не принадлежат одному и тому же человеку. Более того, фактическая стоимость платежа не `19,000 сатоши`, а `4,000 сатоши`. Таким образом, анализ внешнего наблюдателя направлен к ошибочному выводу, обеспечивая сохранение конфиденциальности участников.![схема транзакции payjoin](assets/en/1.webp)
Если вы хотите проанализировать реальную транзакцию Payjoin, вот одна, которую я выполнил на тестнете: [8dba6657ab9bb44824b3317c8cc3f333c2f465d3668c678691a091cdd6e5984c](https://mempool.space/fr/testnet/tx/8dba6657ab9bb44824b3317c8cc3f333c2f465d3668c678691a091cdd6e5984c)
[**-> Ознакомьтесь с нашим учебником о том, как совершить Payjoin с помощью кошелька Samourai**](https://planb.network/tutorials/privacy/payjoin-samourai-wallet)  

[**-> Ознакомьтесь с нашим учебником о том, как совершить Payjoin с помощью кошелька Sparrow**](https://planb.network/tutorials/privacy/payjoin-sparrow-wallet)


**Внешние ресурсы:**
- https://docs.samourai.io/en/spend-tools#stowaway;
- https://gist.githubusercontent.com/LaurentMT/e758767ca4038ac40aaf/raw/c8125f6a3c3d0e90246dc96d3b603690ab6f1dcc/gistfile1.txt;
- https://github.com/bitcoin/bips/blob/master/bip-0078.mediawiki.