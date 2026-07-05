# 15 — WB: корректировки и нормализация adjustments

Этот файл описывает найденные корректировки Wildberries, нормализацию `wb_adjustments`, этапы `before_forpay`, `logistics`, `after_forpay`, `report_level`, важные поля `spp`, `ppvzReward`, `cashbackDiscount`, `rebillLogisticCost`, `deduction`, `paidStorage`, `penalty`, `additionalPayment` и правила, которые нельзя нарушать при расчёте unit economics.

Файл предназначен для другого ИИ-агента. Его нужно читать перед любыми изменениями в WB-нормализации, `raw_json`, формулах forPay, корректировках до forPay, расходах после forPay, report-level расходах и таблице `wb_adjustments`.

---

## 1. Зачем понадобилась нормализация корректировок

В WB-финансовых отчётах оказалось, что базовой формулы недостаточно.

Простая формула:

```text
forPay ≈ retailAmount − ppvzSalesCommission − acquiringFee
```

работает не всегда.

У некоторых товаров `forPay` сильно отличается от ожидаемого значения.

Главный пример:

```text
lopata0126
```

Там `forPay / 1` был больше, чем:

```text
цена продажи − комиссия − эквайринг
```

Причина — скрытые / дополнительные корректировки WB.

---

## 2. Главная цель `wb_adjustments`

Цель таблицы `wb_adjustments`:

```text
нормализовать денежные корректировки WB;
не терять поля из raw_json;
объяснять расхождения до forPay;
объяснять расходы после forPay;
видеть report-level расходы;
не смешивать разные типы денег в одну кучу.
```

То есть `wb_adjustments` — это не просто дубль строк отчёта, а слой объяснения денег.

---

## 3. Почему нельзя полагаться только на нормализованные колонки

WB может менять структуру отчётов.

Часть важных полей сначала находилась только в `raw_json`.

Ранее уже была ошибка:

```text
forPay, vendorCode, deliveryService были в raw_json, но не попадали в нормализованные колонки.
```

После исправления данные стали корректнее.

Правило:

```text
raw_json сохранять всегда;
при появлении новых полей искать их в raw_json;
не удалять raw_json;
не считать unknown money fields безопасными без проверки.
```

---

## 4. Основные stages корректировок

В `wb_adjustments` используется смысловое разделение на этапы:

```text
before_forpay
logistics
after_forpay
report_level
```

Каждый этап отвечает на свой вопрос.

---

## 5. before_forpay

`before_forpay` — корректировки, которые помогают объяснить разницу между:

```text
retailAmount − commission − acquiring
```

и фактическим:

```text
forPay
```

Примеры:

```text
spp_compensation
ppvz_reward
cashback_discount
kvw
kvw_base
vw
vw_nds
commission
acquiring
```

Важно:

```text
before_forpay влияет на объяснение forPay;
его нельзя повторно вычитать после forPay.
```

---

## 6. logistics

`logistics` — логистика WB.

Примеры:

```text
forward_logistics
reverse_logistics
logistics_rebill
```

Источники:

```text
deliveryService
rebillLogisticCost
returnAmount как количество/признак обратной логистики
```

Важно:

```text
deliveryService — деньги;
rebillLogisticCost — деньги;
deliveryAmount — не деньги;
returnAmount — не деньги.
```

---

## 7. after_forpay

`after_forpay` — расходы или доплаты после forPay, которые влияют на остаток после WB.

Примеры:

```text
storage
acceptance
penalty
additional_payment
```

Также некоторые расходы могут быть после forPay, но не всегда привязаны к товарной строке.

---

## 8. report_level

`report_level` — расходы и суммы на уровне отчёта, которые нельзя честно привязать к конкретному товару без отдельной методики.

Примеры:

```text
deduction
bankPaymentSum
часть paidStorage
часть paidAcceptance
часть penalty
```

Главное правило:

```text
report-level расходы не распределять по товарам автоматически без отдельного режима.
```

---

## 9. Важные найденные суммы v164

В аудите v164 были найдены крупные денежные поля:

```text
spp: 73 346 ₽
ppvzReward: 55 054 ₽
cashbackDiscount: 1 241 ₽
kvw: -13 009 ₽
kvwBase: 60 337 ₽
vw: -83 594 ₽
vwNds: -18 391 ₽
deliveryService: 423 580 ₽
rebillLogisticCost: 36 732 ₽
paidStorage: 25 612 ₽
deduction: 340 592 ₽
penalty: 50 ₽
additionalPayment: 128 ₽
```

Особенно важно:

```text
deduction около 340 592 ₽ — крупное удержание WB.
```

Если его не учитывать, экономика может быть завышена.

Если распределить неправильно, экономика товаров может быть искажена.

---

## 10. v165: создана таблица `wb_adjustments`

В v165 была создана таблица:

```text
wb_adjustments
```

Также добавлены нормализованные колонки в:

```text
wb_sales_report_rows
```

Обработано:

```text
15455 rows
50370 adjustment rows
```

---

## 11. Суммы по stages из v165

Зафиксировано:

```text
before_forpay:
  37663 rows
  151731.66 ₽

logistics:
  12488 rows
  460311.56 ₽

after_forpay:
  179 rows
  25661.73 ₽

report_level:
  40 rows
  340719.27 ₽
```

---

## 12. Суммы по kind из v165

Зафиксировано:

```text
spp_compensation: 73345.82 ₽
ppvz_reward: 55054.24 ₽
cashback_discount: 1241.05 ₽
kvw: -13008.52 ₽
kvw_base: 60337.30 ₽
vw: -83594.26 ₽
vw_nds: -18390.85 ₽
commission: -8359.89 ₽
acquiring: 85106.77 ₽
forward_logistics: 376063.28 ₽
reverse_logistics: 47516.27 ₽
logistics_rebill: 36732.01 ₽
storage: 25611.53 ₽
deduction: 340591.60 ₽
penalty: 50.20 ₽
additional_payment: 127.67 ₽
```

Эти цифры важны как контрольная точка.

---

## 13. Report-level расходы v165

Отдельно зафиксировано:

```text
deduction:
  39 строк
  340591.60 ₽

storage:
  156 строк
  25611.53 ₽
```

Важно:

```text
часть расходов не распределена по товарам.
```

Нельзя делать вид, что unit economics полностью точная, если крупные report-level расходы не распределены.

---

## 14. Unknown money fields

После SKIP_FIELDS в v165 было:

```text
Unknown money fields = 0
```

Это хороший результат, но не вечная гарантия.

WB может добавить новые поля.

После новых загрузок нужно снова проверять:

```text
unknown money fields;
новые ключи raw_json;
новые денежные поля;
новые operation types.
```

---

## 15. Проблемы / противоречия v165

В отчёте v165 были противоречия, которые нельзя игнорировать.

Пример:

```text
massagor AfterWB / 1 = 337 ₽
и одновременно "не изменился 354 ₽"
```

Это не может быть верно одновременно.

Ещё пример:

```text
lopata0126 before-forPay adjustment / 1 = -153 ₽
```

Хотя ранее была найдена корректировка примерно:

```text
+333 ₽
```

Это указывает на возможную проблему знаков или формулы.

Вывод:

```text
v165 принял нормализацию как слой данных,
но не как полностью финальную экономику.
```

---

## 16. Знаки корректировок

Знаки — критичная часть.

Для UI:

```text
расходы показываются со знаком минус;
доплаты показываются со знаком плюс;
корректировка до forPay может быть плюс или минус;
forPay должен сходиться с retail − commission − acquiring + correction.
```

Нельзя просто брать абсолютные значения.

Нельзя одновременно хранить расход положительным и показывать его как расход без ясной логики.

---

## 17. Проверка формулы до forPay

Для каждой товарной строки или агрегата нужно проверять:

```text
expected_forpay =
  retailAmount
  − ppvzSalesCommission
  − acquiringFee
  + correction_to_forpay
```

И сравнивать с:

```text
actual_forpay
```

Если разница есть:

```text
показывать расхождение;
искать новые поля raw_json;
не подгонять вручную.
```

---

## 18. lopata0126 как контрольный товар

`lopata0126` — важный контрольный кейс.

Было найдено:

```text
Цена продажи / 1: 1212 ₽
Комиссия / 1: −150 ₽
Эквайринг / 1: −48 ₽
Ожидаемый forPay без корректировки: 1014 ₽
Фактический forPay / 1: 1346 ₽
Корректировка до forPay / 1: +333 ₽
```

Если после изменения кода `lopata0126` снова показывает отрицательную корректировку около `-153 ₽`, нужно проверять знаки.

---

## 19. massagor как контрольный товар

`massagor` — основной товар пользователя.

После исправлений для massagor 30d было:

```text
Выкуплено: 177
Цена продажи / 1: 513 ₽
Комиссия / 1: −24 ₽
Эквайринг / 1: −21 ₽
forPay / 1: 467 ₽
Логистика / 1: −106 ₽
Обратная логистика / 1: −7 ₽
Остаток после WB / 1: 354 ₽
```

Если после нормализации появляется:

```text
337 ₽
```

или другая цифра, нужно объяснить:

```text
что именно добавилось;
какие расходы включены;
нет ли двойного учёта;
нет ли неправильного распределения report-level расходов.
```

---

## 20. Опасность двойного учёта

Нельзя дважды учитывать одно и то же поле.

Особенно опасно:

```text
комиссия;
эквайринг;
deliveryService;
rebillLogisticCost;
deduction;
storage;
additionalPayment.
```

Пример ошибки:

```text
forPay уже включает комиссию и эквайринг,
но код ещё раз вычитает комиссию и эквайринг после forPay.
```

Это неверно.

---

## 21. Эквайринг

`acquiringFee` уже учтён в forPay.

Правило:

```text
показывать эквайринг до forPay;
после forPay повторно не вычитать.
```

Правильная цепочка:

```text
Цена продажи
− Комиссия
− Эквайринг
+/- Корректировка
= forPay
− Логистика
− Прочие расходы после forPay
= Остаток после WB
```

---

## 22. Комиссия WB

Комиссия WB также учитывается до forPay.

Показывать:

```text
− Комиссия WB / 1
```

Не вычитать повторно после forPay.

---

## 23. Логистика

Логистика после forPay.

Поля:

```text
deliveryService
rebillLogisticCost
```

В UI:

```text
− Логистика WB / 1
− Обратная логистика WB / 1
```

Не использовать `deliveryAmount` как рубли.

---

## 24. Доставка и возврат: количество против денег

Критичное правило:

```text
deliveryAmount — не рубли.
returnAmount — не рубли.
```

Если другой ИИ увидит `Amount`, он может ошибочно посчитать это деньгами.

Нужно явно помнить:

```text
deliveryService — деньги логистики;
deliveryAmount — количество / показатель;
returnAmount — количество / показатель обратного движения.
```

---

## 25. deduction

`deduction` — одно из самых опасных полей.

Причины:

```text
может быть крупным;
может быть report-level;
может не относиться к конкретному товару;
если его игнорировать — прибыль завышается;
если распределить неправильно — товарная аналитика искажается.
```

Текущее правило:

```text
deduction показывать отдельно, если report-level;
не распределять по товарам без отдельной методики.
```

---

## 26. paidStorage

`paidStorage` может быть:

```text
строчным расходом;
report-level расходом.
```

Если нет надёжной привязки к товару:

```text
показывать как нераспределённый расход.
```

Не размазывать скрыто.

---

## 27. additionalPayment

`additionalPayment` — доплата.

В UI показывать как плюс:

```text
+ Доплаты
```

В формуле после forPay она увеличивает остаток.

---

## 28. `wb_adjustments` как слой аудита

Таблица `wb_adjustments` должна помогать ответить:

```text
почему forPay такой;
какие расходы пришли после forPay;
какие расходы report-level;
какие поля ещё не учтены;
нет ли двойного учёта.
```

Она не должна быть скрытой магией.

---

## 29. Что желательно показывать в diagnostics

В `/wb/diagnostics` желательно показывать:

```text
количество adjustment rows;
суммы по stage;
суммы по kind;
unknown money fields;
top report-level expenses;
rows with large correction_to_forpay;
товары с biggest discrepancy;
поля raw_json, которые ещё не нормализованы.
```

---

## 30. Что желательно показывать в field dictionary

В `/wb/field-dictionary` для каждого adjustment-kind желательно указать:

```text
source raw field;
stage;
is_money;
sign convention;
used_in_forpay;
used_after_forpay;
report_level или row_level;
UI label;
warning.
```

---

## 31. Что делать при новых WB-полях

Если WB добавил новое поле:

```text
сначала найти его в raw_json;
понять денежное оно или нет;
добавить в diagnostics;
добавить в field dictionary;
решить stage;
решить sign convention;
добавить test/check;
только потом включать в формулы.
```

Нельзя сразу добавлять поле в прибыль без проверки.

---

## 32. Проверка знаков

Для каждого kind нужно знать знак.

Примерная UI-логика:

```text
commission → расход → минус
acquiring → расход → минус
spp_compensation → может быть плюс
ppvz_reward → может быть плюс/корректировка
cashback_discount → зависит от смысла поля
forward_logistics → расход → минус
reverse_logistics → расход → минус
storage → расход → минус
deduction → расход → минус
penalty → расход → минус
additional_payment → доплата → плюс
```

Если знак неясен, не включать молча в итог.

---

## 33. Проверка агрегатов

После изменений нужно сверять:

```text
sum(adjustments by stage)
sum(adjustments by kind)
sum(row normalized fields)
sum(report fields)
forPaySum
bankPaymentSum
расхождение formulas
```

Не принимать расчёт только потому, что страница открылась.

---

## 34. Проверка UI после adjustments

После изменений UI должен показывать:

```text
+/- Корректировка до forPay / 1;
forPay / 1;
− Логистика WB / 1;
− Обратная логистика WB / 1;
− Расходы WB после forPay / 1;
Остаток после WB / 1;
предупреждение о report-level расходах, если есть.
```

---

## 35. Report-level warning

Если есть нераспределённые report-level расходы, UI должен предупреждать:

```text
Часть расходов WB находится на уровне отчёта и не распределена по товарам.
Остаток после WB по товарам может быть выше фактической экономики.
```

Не нужно пугать, но нужно быть честным.

---

## 36. Что нельзя делать

Нельзя:

```text
удалять raw_json;
скрывать unknown money fields;
игнорировать deduction;
считать deliveryAmount рублями;
считать returnAmount рублями;
повторно вычитать commission/acquiring после forPay;
переворачивать знаки без проверки;
распределять report-level расходы без отдельного режима;
называть forPay чистой прибылью;
называть остаток после WB чистой прибылью;
принимать v165 как полностью финальную экономику без reconciliation.
```

---

## 37. Проверки после изменений

Проверить:

```text
/wb открывается;
/wb/diagnostics открывается;
/wb/field-dictionary открывается;
/wb/calculation-rules открывается;
/wb/money-flow открывается;
/wb/products открывается;
/wb/products/{nm_id} открывается;
/wb/unit-economics открывается;
GET не вызывает WB API;
massagor не сломан;
lopata0126 показывает корректировку правильно;
unknown money fields проверены;
report-level deduction не размазан молча;
расходы отображаются с минусом.
```

---

## 38. Browser screenshots

Для задач по adjustments нужны screenshots:

```text
/tmp/vXXX_wb_unit_economics_adjustments.png
/tmp/vXXX_wb_product_lopata0126_adjustment.png
/tmp/vXXX_wb_product_massagor_adjustment.png
/tmp/vXXX_wb_diagnostics_adjustments.png
/tmp/vXXX_wb_field_dictionary_adjustments.png
```

---

## 39. Что писать в report_vXXX.md

Если задача меняет adjustments, report должен содержать:

```text
что изменено;
какие поля raw_json добавлены;
какие stages/kinds добавлены;
какие суммы по stages;
какие суммы по kinds;
unknown money fields;
проверка знаков;
проверка massagor;
проверка lopata0126;
что с deduction;
что с report-level расходами;
есть ли двойной учёт;
GET no WB API;
screenshots;
checks;
git commit и tag.
```

---

## 40. История версий

Связанные этапы:

```text
v163 — найдена корректировка до forPay по lopata0126
v164 — аудит скрытых корректировок WB
v165 — создана нормализация wb_adjustments
```

Теги:

```text
v1.61-wb-forpay-adjustment-reconciliation
v1.62-wb-hidden-adjustments-audit
v1.63-wb-adjustments-normalization
```

---

## 41. Текущая точка

На момент handoff:

```text
слой wb_adjustments создан;
важные корректировки найдены;
unknown money fields были сведены к 0;
deduction найден как крупный report-level расход;
но финальная формула чистой прибыли ещё не сделана;
нужен будущий reconciliation по формулам и знакам.
```

---

## 42. Главное резюме

Кратко:

```text
wb_adjustments нужен, чтобы объяснять деньги WB.
before_forpay объясняет forPay.
logistics — логистика.
after_forpay — расходы после forPay.
report_level — расходы уровня отчёта, не привязанные честно к товару.
forPay не чистая прибыль.
Остаток после WB не чистая прибыль.
deliveryAmount и returnAmount не рубли.
deduction около 340 591 ₽ — критичное удержание.
Report-level расходы не распределять без отдельного режима.
massagor и lopata0126 — обязательные контрольные товары.
```
