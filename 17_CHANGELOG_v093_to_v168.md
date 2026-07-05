# 17 — Changelog v093–v168: история ключевых изменений DVRK Control

Этот файл фиксирует важную историю изменений проекта DVRK Control от этапа очистки dev-базы и стабилизации заказов до WB-аналитики, UI-настроек, `/logs` и `/integrations`.

Файл предназначен для другого ИИ-агента. Его нужно читать, чтобы понимать, почему система устроена именно так, какие решения уже были приняты, какие задачи были частично приняты, а какие нельзя повторять как ошибки.

---

## 1. Для чего нужен этот changelog

Этот файл не является полным git log.

Его цель:

```text
дать другому ИИ исторический контекст;
объяснить, почему появились текущие правила;
показать, какие ошибки уже были найдены и исправлены;
зафиксировать принятые решения;
показать текущую точку проекта.
```

Если другой ИИ будет работать без этого контекста, он может случайно вернуть старые ошибки.

---

## 2. Общая линия развития проекта

Проект развивался по направлениям:

```text
заказы Tilda;
YooKassa и второй закрывающий чек;
Честный Знак через ZH в SKU;
карточки заказов и /chz;
статистика и CRM;
логи и backup;
WB-аналитика;
WB unit economics;
WB склады;
WB корректировки;
UI-настройки;
страница /integrations.
```

---

## 3. v093 — очистка dev-базы и тестовые заказы

В районе v093 dev-база была очищена от старых тестовых заказов.

Важно:

```text
продукты и справочники были оставлены;
старые тестовые заказы убраны;
позже был добавлен фильтр по датам, поэтому отдельная очистка/архив тестовых заказов стала менее актуальной.
```

Вывод:

```text
не нужно снова поднимать задачу очистки тестовых заказов без причины.
```

---

## 4. Стабилизация Tilda / YooKassa / ЧЗ

Была зафиксирована главная бизнес-логика:

```text
Tilda принимает оплату.
Tilda / YooKassa формируют первый чек предоплаты.
DVRK Control формирует второй закрывающий чек.
```

ЧЗ определяется через SKU:

```text
ZH
```

Это критичная стабильная логика.

---

## 5. Первый чек и второй чек

Принято:

```text
первый чек делает Tilda / YooKassa;
DVRK Control не делает первый чек;
DVRK Control делает только второй / закрывающий чек.
```

Если Tilda/YooKassa прислали:

```text
full_prepayment → закрывающий чек нужен;
full_payment → закрывающий чек не нужен;
unknown → проверить признак расчёта.
```

---

## 6. YooKassa payload

Было учтено:

```text
ФФД 1.2;
mark_code_info;
gs_1m;
код ЧЗ не кодировать Base64;
payment_subject marked для ZH;
payment_subject commodity для обычного товара;
payment_subject service для доставки;
payment_mode full_payment в закрывающем чеке;
settlements type prepayment.
```

---

## 7. Защита от дублей YooKassa

Принято:

```text
нельзя отправлять повторный закрывающий чек, если status succeeded;
нельзя отправлять повторный чек, если status pending;
нужен idempotency key;
page load не вызывает YooKassa;
polling только для pending/sent;
примерный polling interval 10 секунд.
```

---

## 8. /orders

`/orders` стал рабочей очередью заказов.

Основные колонки:

```text
ID;
delete;
Tilda ID;
покупатель;
телефон;
email;
адрес;
доставка;
статус;
ЧЗ;
позиции;
сумма;
дата;
действия.
```

Покупатель отображается по приоритету:

```text
delivery_fio
↓
customer_name
↓
пусто
```

---

## 9. /orders/{id}

Карточка заказа стала показывать:

```text
Покупатель;
Доставка;
Оплата;
YooKassa;
Товары / позиции;
Быстрые действия;
финансовую расшифровку.
```

Финансы:

```text
Товары до скидки;
Скидка / промокод;
Товары после скидки;
Доставка;
Итого оплачено.
```

---

## 10. /chz

`/chz` стал очередью задач по Честному Знаку и закрывающим чекам.

Статусы:

```text
Нужен код ЧЗ;
Готов к закрывающему чеку;
Закрывающий чек без ЧЗ;
Чек не нужен;
Проверить признак расчёта.
```

---

## 11. /chz/orders/{id}

Карточка ЧЗ / закрывающего чека стала рабочим местом оператора.

Принято:

```text
preview закрывающего чека;
сверка сумм;
ввод кода ЧЗ;
checkbox подтверждения;
confirm перед отправкой;
backend повторно проверяет безопасность;
audit events.
```

---

## 12. Скидки и промокоды

Была стабилизирована финансовая модель:

```text
goods_gross_sum;
goods_discount_sum;
goods_net_sum;
delivery_sum;
order_total_paid.
```

Формула:

```text
goods_net_sum + delivery_sum = order_total_paid
```

Скидка распределяется только между товарами.

---

## 13. Пример заказа #506

Для заказа #506:

```text
5010_CLUB Косметичка
gross: 299.00
promo: DARK2610
discount: 29.90
net: 269.10
delivery: 1053.03
total: 1322.13
```

Этот пример использовался для проверки скидок.

---

## 14. Пример заказа #517

Для заказа #517:

```text
5009_CLUB Кепка K2
gross: 299.00
promo: DARK2609
discount: 29.90
net: 269.10
delivery: 1078.28
total: 1347.38
```

По #517 был успешный non-ZH closing receipt.

---

## 15. SKU, секции и CLUB

Пользователь объяснил правила:

```text
1xxx = Мужское
2xxx = Женское
5xxx = Аксессуары
ZH = Честный Знак
CLUB = клуб / акция
```

Принято:

```text
кепка или жилет в аксессуарах — не ошибка;
размер у аксессуара — не ошибка;
система не спорит с каталогом пользователя.
```

---

## 16. /stats

`/stats` стал управленческой статистикой.

Блоки:

```text
сводные карточки;
продажи по товарам;
финансы;
бизнес-метрики;
ЧЗ;
география;
динамика;
месяцы;
ассортимент;
структура финансов;
размеры;
товары/заказы по секциям;
CLUB и секции;
контроль каталога;
промокоды и скидки.
```

---

## 17. Контроль каталога

Было принято убрать ложные проверки:

```text
Одежда в аксессуарах;
Размер у аксессуара;
Кепка в аксессуарах как проблема;
Жилет в аксессуарах как проблема.
```

Оставить только объективные проверки:

```text
SKU без секции;
CLUB без базовой секции;
одежда без размера, если применимо.
```

---

## 18. /customers

Создана CRM-страница покупателей.

Правила:

```text
email — главный ключ;
phone — второй ключ;
не склеивать по имени;
не определять пол по имени;
Мужское/Женское — секции товара, не пол покупателя;
маркетинг только при согласии Marketing.
```

---

## 19. Согласия покупателей

Учитываются:

```text
Oferta_and_Soglashenia;
Person_data;
Marketing.
```

Система сама ничего не отправляет клиентам.

Marketing можно использовать только при наличии согласия.

---

## 20. /logs до v168

До v168 `/logs` был слишком длинным и неудобным.

Проблемы:

```text
длинное полотно;
сложно быстро найти важное;
backup и события смешаны;
не хватало профессиональной структуры.
```

---

## 21. backup

Была настроена backup-система.

Backup directory:

```text
/opt/dvrk_backups/daily/
```

Windows pull backup:

```text
C:\DVRK_BACKUPS
C:\Users\mrdob\.ssh\dvrk_backup_ed25519
```

Первый успешно скачанный файл:

```text
dvrk_full_backup_2026-07-02_17-18-25.tar.gz
```

---

## 22. WB-модуль: начало

Пользователь попросил понять, продаёт ли он конкретный товар на WB в плюс или минус.

Главная цель:

```text
получить экономику конкретного товара;
понять, сколько WB начисляет за 1 реализованный товар;
потом добавить себестоимость и чистую прибыль.
```

---

## 23. WB API

Принято использовать финансовые отчёты WB.

Основной API:

```text
POST /api/finance/v1/sales-reports/detailed
```

Список отчётов:

```text
POST /api/finance/v1/sales-reports/list
```

Token category:

```text
Финансы
```

---

## 24. WB GET no API

Принято:

```text
GET /wb* не вызывает WB API.
```

WB API только через ручную синхронизацию/action.

Cooldown:

```text
минимум 10 минут;
лучше 1 час.
```

---

## 25. v149 — WB field dictionary

Tag:

```text
v1.47-wb-field-dictionary
```

Сделано:

```text
/wb/field-dictionary;
/wb/calculation-rules;
/wb/diagnostics;
wb_field_dictionary;
70 WB-полей;
89 raw_json ключей;
71% покрытие.
```

Критично:

```text
deliveryAmount и returnAmount — не рубли.
```

---

## 26. v151 — period filter fix

Tag:

```text
v1.49-wb-period-filter-fix
```

Исправлено:

```text
периоды реально фильтруют данные;
периоды считаются от последней фактической даты операции WB;
если WB данные отстают, не показывать пустые текущие дни.
```

---

## 27. v152 — WB June normalization fix

Tag:

```text
v1.50-wb-june-normalization-fix
```

Проблема:

```text
forPay, vendorCode, deliveryService были в raw_json, но не нормализовались.
```

После исправления:

```text
forPay начал распознаваться;
артикулы вернулись;
deliveryService стал доступен.
```

---

## 28. v153 — expenses dates reconcile

Tag:

```text
v1.51-wb-expenses-dates-reconcile
```

Зафиксирована формула WB расходов:

```text
комиссия
+ логистика
+ возвратная логистика
+ эквайринг
+ хранение
+ приёмка
+ удержания
+ штрафы
− доплаты
```

---

## 29. v154 — real sales classification

Tag:

```text
v1.52-wb-real-sales-classification
```

Ошибка:

```text
старое "Продано" считало SUM(quantity) по всем товарным строкам,
включая логистику и возмещения.
```

Исправлено:

```text
Продажа / Продажа → redeemed_sale;
Логистика → logistics;
Возврат / Возврат → customer_return_after_redeem;
Хранение → storage;
Штраф → penalty;
Удержание → deduction;
Компенсация лояльности → additional_payment.
```

---

## 30. v155 — money flow forPay to payout

Tag:

```text
v1.53-wb-money-flow-forpay-payout
```

Принято:

```text
forPay — начисление по товарной строке;
bankPaymentSum — итог на уровне отчёта;
forPay не финальный банковский перевод;
эквайринг уже учтён в forPay.
```

---

## 31. v156 — commission and nonbuyout logic

Tag:

```text
v1.54-wb-commission-nonbuyout-logic
```

Принято по комиссии:

```text
retailAmount − ppvzSalesCommission − acquiringFee ≈ forPay
```

Комиссию повторно вычитать нельзя.

Но nonbuyout UI тогда ещё показывал 0, хотя в данных были строки returnAmount > 0.

---

## 32. v157 — nonbuyout UI and reconcile diff

Tag:

```text
v1.55-wb-nonbuyout-reconcile-diff
```

Исправлено:

```text
nonbuyout считать через returnAmount, а не quantity.
```

Пример massagor 30d:

```text
Выкуплено: 177
Невыкуп: 24
forPay / 1: 467 ₽
Расходы после forPay / 1: 113 ₽
Остаток после WB / 1: 354 ₽
```

---

## 33. v158 — forPaySum vs bankPaymentSum

Tag:

```text
v1.56-wb-forpay-bankpayment-semantics
```

Исправлено разделение:

```text
forPaySum = начислено по отчётам;
bankPaymentSum = итого к оплате по отчётам.
```

Зафиксировано:

```text
bankPaymentSum нельзя считать точной выплатой за товар.
```

---

## 34. v159 — logistics cost columns

Tag:

```text
v1.57-wb-logistics-cost-columns
```

Добавлены:

```text
Логистика WB / 1;
Обратная логистика WB / 1;
Расходы WB после forPay / 1;
Остаток после WB / 1.
```

`Расчётно после WB` переименовано в:

```text
Остаток после WB
```

---

## 35. v160 — warehouse logistics

Tag:

```text
v1.58-wb-warehouse-logistics
```

Созданы:

```text
/wb/warehouses;
/wb/warehouses/{warehouse};
блок "Логистика по складам" в карточке товара.
```

Найдены особенности данных:

```text
warehouseName / warehouseId заполнены 0%;
officeName часто используется как источник склада;
ppvzOfficeName — не всегда склад.
```

---

## 36. v161 — negative expense display

Tag занят:

```text
v1.59-wb-negative-expense-display
```

Сделано:

```text
расходы WB отображаются со знаком минус;
forPay объясняется как начисление после комиссии и эквайринга;
forPay не чистая прибыль.
```

---

## 37. v162 — product warehouse logistics

Tag:

```text
v1.60-wb-product-warehouse-logistics
```

Создано:

```text
/wb/product-warehouses
```

Логика:

```text
товар → склады
```

Пользователь хотел именно это дополнение к “склад → товары”.

---

## 38. v163 — forPay adjustment reconciliation

Tag:

```text
v1.61-wb-forpay-adjustment-reconciliation
```

Найдено, что для `lopata0126` forPay объясняется корректировкой.

Пример:

```text
Цена продажи / 1: 1212 ₽
Комиссия / 1: −150 ₽
Эквайринг / 1: −48 ₽
Ожидаемый forPay: 1014 ₽
Фактический forPay / 1: 1346 ₽
Корректировка: +333 ₽
```

Добавлена идея:

```text
+/- Корректировка до forPay / 1
```

---

## 39. v164 — hidden adjustments audit

Tag:

```text
v1.62-wb-hidden-adjustments-audit
```

Найдены поля:

```text
spp;
ppvzReward;
cashbackDiscount;
kvw;
kvwBase;
vw;
vwNds;
deliveryService;
rebillLogisticCost;
paidStorage;
deduction;
penalty;
additionalPayment.
```

Критично:

```text
deduction около 340 592 ₽.
```

---

## 40. v165 — adjustments normalization

Tag:

```text
v1.63-wb-adjustments-normalization
```

Создано:

```text
wb_adjustments
```

Стадии:

```text
before_forpay;
logistics;
after_forpay;
report_level.
```

Важно:

```text
v165 принят как слой нормализации,
но не как финальная экономика,
потому что в отчёте были противоречия по знакам и итогам.
```

---

## 41. v166 — formula reconciliation

Была предложена задача v166 по reconciliation формул WB, но пользователь решил пока закончить WB.

```text
v166 formula reconciliation не запускался.
```

Не считать его выполненным.

---

## 42. v167 — stats/customers UI settings

Tag:

```text
v1.64-stats-customers-ui-settings
```

Создана таблица:

```text
user_ui_settings
```

Разрешённые страницы:

```text
stats
customers
```

Сохраняется:

```text
/stats:
  period;
  custom period;
  collapsed sections;
  section order;
  metric cards.

/customers:
  search;
  period;
  segment;
  section;
  marketing filter.
```

URL params имеют приоритет.

---

## 43. v168 — logs + integrations settings

Tag:

```text
v1.65-logs-integrations-settings
```

Сделано:

```text
/logs redesigned as compact dashboard;
/integrations created;
integration_settings;
integration_settings_audit;
YooKassa/Tilda/common settings;
secret masking;
empty secret does not overwrite;
GET no external API.
```

Принято:

```text
/logs как хорошая база.
```

Частично принято:

```text
/integrations создана, но требует доработки.
```

---

## 44. /integrations после v168

Есть:

```text
YooKassa block;
Tilda block;
common settings;
audit history;
masked secrets.
```

Нужно доработать:

```text
YooKassa API URL;
callback/webhook URL;
last response;
last receipt;
last error;
Tilda Public key;
Project/Site ID;
Form ID;
critical Tilda settings;
кнопка "Открыть инструкцию".
```

Логичная следующая задача:

```text
v169 — расширить /integrations.
```

---

## 45. Уведомления

Обсуждались email / Telegram уведомления.

Пользователь решил:

```text
не делать уведомления.
```

Не создавать задачу по уведомлениям без нового запроса.

---

## 46. Текущая стабильная точка

На момент handoff:

```text
Tilda webhook работает;
заказы принимаются;
ЧЗ через ZH работает;
второй закрывающий чек реализован;
скидки учитываются;
статистика и CRM работают;
backup и logs работают;
WB-модуль сильно развит;
UI settings для stats/customers работают;
/integrations создана, но требует расширения.
```

Последний важный tag:

```text
v1.65-logs-integrations-settings
```

---

## 47. Что другому ИИ нельзя забыть

Главные исторические ошибки, которые нельзя повторять:

```text
считать deliveryAmount рублями;
считать returnAmount рублями;
считать все quantity продажами;
называть forPay чистой прибылью;
повторно вычитать эквайринг после forPay;
игнорировать deduction;
распределять report-level расходы без методики;
требовать ЧЗ для обычных товаров;
не требовать ЧЗ для ZH;
отправлять второй чек при full_payment;
вызывать внешние API на GET.
```

---

## 48. Главное резюме

Кратко:

```text
v093 — очистка dev-заказов.
Дальше стабилизированы Tilda, YooKassa, ЧЗ и закрывающий чек.
Затем развиты /orders, /chz, /stats, /customers.
WB-модуль прошёл несколько важных исправлений: нормализация forPay, классификация реальных продаж, nonbuyout, bankPaymentSum, логистика, склады, корректировки.
v167 — сохранение UI settings только для /stats и /customers.
v168 — компактный /logs и новая /integrations.
Текущая точка: v1.65-logs-integrations-settings.
Следующий логичный шаг: v169 расширить /integrations.
```
