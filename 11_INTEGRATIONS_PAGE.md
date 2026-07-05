# 11 — Страница `/integrations`: YooKassa, Tilda и настройки подключений

Этот файл описывает страницу `/integrations` в DVRK Control: зачем она создана, какие настройки там должны быть, как хранить secrets, как отображать YooKassa и Tilda, что уже сделано в v168 и что нужно доработать дальше.

Файл предназначен для другого ИИ-агента. Его нужно читать перед любыми изменениями в `/integrations`, настройках YooKassa, настройках Tilda, webhook URL, secret key, token masking, проверках подключений и истории изменений интеграций.

---

## 1. Зачем нужна `/integrations`

`/integrations` — это отдельная страница для подключений внешних сервисов.

Основные сервисы:

```text
YooKassa
Tilda
```

В будущем туда можно добавлять другие интеграции, но только аккуратно и без смешивания логики.

Страница нужна, чтобы пользователь мог:

```text
видеть текущие настройки подключения;
сохранять Shop ID / tokens / secrets;
проверять подключение;
видеть webhook URL;
копировать webhook URL;
видеть критические настройки Tilda;
понимать, что работает, а что требует внимания.
```

---

## 2. Почему `/integrations` вынесли отдельно

До этого настройки были разбросаны.

Пользователь попросил создать нормальную страницу:

```text
Подключение
```

Идея:

```text
все настройки внешних сервисов должны быть в одном месте;
не искать их по разным страницам;
не хранить подключение в коде без UI;
не показывать secrets открыто.
```

---

## 3. Текущая точка после v168

В v168 была создана страница:

```text
/integrations
```

Также были созданы таблицы:

```text
integration_settings
integration_settings_audit
```

Сделано:

```text
YooKassa settings block;
Tilda settings block;
common settings;
audit history;
secret masking;
empty secret does not overwrite existing;
GET /integrations не вызывает внешние API;
старые страницы не сломаны.
```

v168 принят как рабочая база, но `/integrations` ещё не считается финально завершённой.

---

## 4. Что было принято в v168

Принято:

```text
страница /integrations существует;
страница открывается;
настройки можно сохранять;
secrets masked;
secrets не возвращаются полностью через API;
пустой secret не затирает сохранённый secret;
есть audit history изменений;
GET /integrations только читает БД;
старые страницы /orders, /chz, /stats, /customers, /wb, /tilda работают.
```

Checks v168:

```text
py_compile OK;
smoke OK;
check_logs_integrations_v168.py 32/32;
browser checks 22/22;
tag: v1.65-logs-integrations-settings.
```

---

## 5. Что не было полностью принято

Страница `/integrations` в v168 была рабочей, но слишком упрощённой.

По YooKassa не хватало:

```text
API URL / endpoint;
callback URL webhook с кнопкой копирования;
авто-проверка webhook;
последний API response;
последний receipt;
webhook OK;
last error;
кнопка "Отправить тест".
```

По Tilda не хватало:

```text
Public key;
Project / Site ID;
Form ID;
send type: after payment only;
products array enabled;
application/json off;
Cookies off;
externalid off;
кнопка "Открыть инструкцию";
более понятная стабильная инструкция по настройкам Tilda.
```

Следующая возможная задача после handoff:

```text
v169 — расширить /integrations до полноценной панели YooKassa/Tilda.
```

---

## 6. Главный принцип безопасности

Secrets нельзя показывать полностью.

К secrets относятся:

```text
YooKassa Secret Key;
Tilda webhook token / secret;
WB token, если его когда-нибудь перенесут сюда;
любые API keys;
Authorization headers;
passwords.
```

Правила:

```text
не отдавать secret полностью через API;
не показывать secret полностью в HTML;
не логировать secret;
не писать secret в audit;
не отправлять secret в browser, если это не нужно;
показывать только masked value.
```

Пример masked:

```text
sk_live_************abcd
```

Или:

```text
••••••••••••abcd
```

---

## 7. Сохранение secrets

Если пользователь сохраняет форму и оставляет secret-поле пустым:

```text
старое значение secret не должно затираться.
```

Это важное правило.

Поведение:

```text
secret field empty → keep existing secret;
secret field filled → replace secret;
non-secret fields update normally.
```

В UI рядом с secret-полем желательно пояснение:

```text
Оставьте поле пустым, чтобы не менять сохранённый ключ.
```

---

## 8. Audit настроек интеграций

Каждое изменение настроек нужно писать в:

```text
integration_settings_audit
```

Но audit не должен содержать secrets.

Можно писать:

```text
какая настройка изменена;
какой сервис;
кто/что изменило;
когда;
старое masked значение;
новое masked значение;
источник действия.
```

Нельзя писать:

```text
полный secret key;
полный token;
Authorization header.
```

---

## 9. GET `/integrations`

GET `/integrations` должен быть безопасным.

Разрешено:

```text
читать локальную БД;
показывать сохранённые настройки;
показывать masked secrets;
показывать последнюю локальную историю;
показывать последние локально сохранённые статусы.
```

Запрещено:

```text
вызывать YooKassa API;
вызывать Tilda API;
проверять webhook внешним запросом;
отправлять тестовый чек;
запускать WB sync;
изменять настройки.
```

Все активные действия должны быть через POST / button action.

---

## 10. YooKassa block

Блок YooKassa должен включать:

```text
Shop ID;
Secret Key;
режим: test / live;
включены ли чеки;
API URL / endpoint;
callback URL / webhook URL;
статус подключения;
последняя проверка;
последний receipt;
последняя ошибка;
кнопка "Сохранить";
кнопка "Проверить подключение";
кнопка "Отправить тест" — только если реализовано безопасно.
```

---

## 11. YooKassa Shop ID

`Shop ID` — не secret, но всё равно не нужно лишний раз логировать.

Показывать можно.

Используется для авторизации в YooKassa API вместе с secret key.

---

## 12. YooKassa Secret Key

`Secret Key` — secret.

Правила:

```text
в UI показывать masked;
в input не подставлять реальный secret;
пустое поле не затирает старый secret;
при новом вводе заменить secret;
не логировать.
```

---

## 13. YooKassa mode

Нужен режим:

```text
test
live
```

Для dev-среды часто используется test mode.

В UI должно быть явно видно:

```text
Тестовый режим
```

или:

```text
Боевой режим
```

Если включён live mode, желательно предупреждение:

```text
Действия могут отправлять реальные запросы в YooKassa.
```

---

## 14. YooKassa receipts toggle

Можно иметь настройку:

```text
Отправка закрывающих чеков включена / выключена
```

Если выключена:

```text
UI должен показывать предупреждение;
кнопки отправки чеков должны учитывать это состояние;
backend обязан повторно проверять.
```

---

## 15. YooKassa callback URL / webhook

На странице нужно показывать URL для callback/webhook, если он используется.

Нужно:

```text
отображать URL;
дать кнопку "Скопировать";
пояснить, куда вставлять в YooKassa;
показывать последний webhook OK / error, если это реализовано.
```

---

## 16. YooKassa last response / last receipt

Полезно показывать:

```text
последний receipt_id;
статус;
время;
связанный order_id;
последнюю ошибку;
краткий safe response.
```

Нельзя показывать sensitive payload.

---

## 17. YooKassa test button

Кнопка "Отправить тест" нужна только если реализована безопасно.

Минимальные правила:

```text
не отправлять реальный чек без явного confirm;
в live mode требовать дополнительное подтверждение;
не создавать чек на GET;
писать audit;
показывать результат;
не логировать secret.
```

Если нет безопасного сценария — кнопку не делать.

---

## 18. Tilda block

Блок Tilda должен включать:

```text
Webhook URL;
Webhook token / secret;
Public key, если используется;
Project / Site ID;
Form ID;
критические настройки;
последний webhook;
последняя ошибка;
кнопка "Сохранить";
кнопка "Проверить webhook";
кнопка "Открыть инструкцию";
кнопка "Скопировать webhook URL".
```

---

## 19. Tilda Webhook URL

Webhook URL должен быть виден и легко копироваться.

Пример назначения:

```text
этот URL вставляется в настройки Tilda, чтобы Tilda отправляла оплаченные заказы в DVRK Control.
```

Нужно показывать:

```text
URL;
copy button;
краткую подсказку.
```

---

## 20. Tilda token / secret

Webhook token / secret — secret.

Правила такие же:

```text
masked в UI;
не отдавать полностью через API;
не логировать;
пустое поле не затирает старый secret;
при новом вводе заменить.
```

---

## 21. Tilda critical settings

В блоке Tilda обязательно показывать стабильные настройки:

```text
API METHOD: Off
Передавать данные по товарам в заказе — массивом: Включено
Отправлять только после оплаты: Включено
application/json: Выключено
Cookies: Выключено
externalid: Выключено
```

Эти настройки критичны для parser webhook.

---

## 22. Почему Tilda settings важны

Если Tilda настроена неправильно, может сломаться:

```text
приём заказов;
товарные позиции;
SKU;
ЧЗ через ZH;
YooKassa payment_id;
суммы;
скидки;
доставка;
второй закрывающий чек.
```

Поэтому на `/integrations` нужно не просто хранить токены, а показывать понятную инструкцию.

---

## 23. Tilda Project / Site ID и Form ID

Эти поля можно хранить для удобства:

```text
Project / Site ID;
Form ID;
Public key.
```

Они нужны, чтобы другой ИИ/оператор понимал, к какому сайту и форме относятся настройки.

Если эти поля пока не используются backend-логикой, это нужно явно указать в UI как справочную настройку.

---

## 24. Tilda check button

Кнопка "Проверить webhook" не должна на GET дергать внешнее API.

POST-проверка может смотреть:

```text
есть ли webhook URL;
есть ли token, если используется;
есть ли последние webhook-события;
последний webhook OK;
последняя ошибка parser;
правильно ли сохранены critical flags.
```

Если будет реализована внешняя проверка Tilda API — только отдельной задачей, с rate limit и audit.

---

## 25. Общие настройки

В `/integrations` может быть общий блок:

```text
название окружения;
dev / prod;
base URL;
режим безопасной отправки;
дата последнего изменения;
кто изменил;
status summary.
```

Но не перегружать страницу.

---

## 26. Интеграционные статусы

Статусы должны быть понятными:

```text
Подключено
Не настроено
Требует проверки
Последняя проверка успешна
Последняя ошибка
Тестовый режим
Боевой режим
Webhook получен
Webhook давно не приходил
```

Не показывать только technical enum.

---

## 27. UI страницы

Страница должна быть:

```text
тёмная;
компактная;
разделена на карточки;
без длинного полотна;
с понятными кнопками;
с подсказками через tooltip;
с явными warning для live mode и неправильных Tilda settings.
```

Оптимальная структура:

```text
верхние summary cards;
карточка YooKassa;
карточка Tilda;
карточка Общие настройки;
карточка История изменений;
блок инструкции.
```

---

## 28. Что нельзя делать на `/integrations`

Нельзя:

```text
показывать secret полностью;
затирать secret пустым полем;
вызывать внешние API на GET;
отправлять тестовый чек без confirm;
смешивать WB token с Tilda/YooKassa без отдельной задачи;
логировать Authorization;
выводить .env;
показывать полный old/new secret в audit;
ломать /tilda, если старая страница ещё существует.
```

---

## 29. Старые страницы

После создания `/integrations` старые страницы не должны ломаться:

```text
/orders
/chz
/stats
/customers
/wb
/tilda
/logs
```

Если `/tilda` больше не основная страница, можно оставить redirect или информационный блок:

```text
Настройки Tilda теперь находятся в /integrations.
```

Но не удалять без проверки.

---

## 30. Связь с `/logs`

`/logs` показывает системные события.

`/integrations` может писать туда события:

```text
YooKassa settings updated;
Tilda settings updated;
secret changed;
connection check OK;
connection check failed;
webhook check OK;
webhook check failed.
```

Secrets в logs запрещены.

---

## 31. Связь с `/orders` и `/chz`

Настройки YooKassa и Tilda напрямую влияют на:

```text
приём заказов;
YooKassa payment_id;
отправку закрывающих чеков;
статус ЧЗ;
receipt status.
```

Поэтому после изменения `/integrations` нужно проверить:

```text
/orders;
/chz;
/chz/orders/{id};
отправка preview не сломана;
GET не вызвал YooKassa.
```

---

## 32. Связь с WB

WB уже имеет свои страницы:

```text
/wb/settings
/wb/sync
/wb/logs
```

Не нужно автоматически переносить WB token в `/integrations` без отдельного решения.

WB API имеет свои rate limits и cooldown.

---

## 33. Проверки после изменений `/integrations`

Обязательно проверить:

```text
/integrations открывается;
YooKassa block виден;
Tilda block виден;
secret masked;
пустой secret не затирает старый;
новый secret сохраняется;
audit создаётся без secret;
GET /integrations не вызывает внешние API;
POST save работает;
POST check работает, если реализован;
ошибки показываются понятно;
старые страницы открываются.
```

---

## 34. Browser screenshots

Для задач по `/integrations` нужны screenshots:

```text
/tmp/vXXX_integrations_main.png
/tmp/vXXX_integrations_yookassa.png
/tmp/vXXX_integrations_tilda.png
/tmp/vXXX_integrations_audit.png
```

Если проверяется masking:

```text
/tmp/vXXX_integrations_masked_secrets.png
```

---

## 35. Что писать в report_vXXX.md

Если задача меняет `/integrations`, report должен содержать:

```text
что изменено;
какие настройки добавлены;
как проверен secret masking;
как проверено empty secret does not overwrite;
как проверен audit без secrets;
как проверено, что GET не вызывает внешние API;
какие страницы не сломаны;
какие screenshots сохранены;
какие checks прошли;
git commit и tag.
```

---

## 36. Возможная следующая задача v169

Логичная следующая задача:

```text
v169 — расширить /integrations до полноценной панели YooKassa/Tilda.
```

Содержимое v169:

```text
не трогать /logs без необходимости;
расширить YooKassa block;
расширить Tilda block;
добавить callback/webhook URL copy;
добавить critical settings Tilda;
добавить last receipt / last webhook / last error;
сохранить secret masking;
сохранить GET no external API;
сделать browser screenshots.
```

---

## 37. Главное резюме

Кратко:

```text
/integrations — страница подключений YooKassa и Tilda.
Создана в v168, но требует дальнейшей доработки.
Secrets показывать только masked.
Пустой secret не затирает старый.
GET /integrations не вызывает внешние API.
Audit изменений не содержит secrets.
Tilda critical settings должны быть явно показаны.
YooKassa должна показывать mode, status, last receipt, last error.
Старые страницы не ломать.
Следующая логичная задача — v169 расширение /integrations.
```
