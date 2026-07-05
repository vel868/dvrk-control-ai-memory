# 20 — Final Package Index and Checklist

Этот файл является финальным индексом и контрольным списком для пакета handoff-файлов DVRK Control.

Его задача — помочь другому ИИ-агенту быстро проверить, что он получил полный комплект документов, в каком порядке их читать, что считать критичным, и какие проверки выполнить перед продолжением разработки.

---

## 1. Назначение пакета

Пакет handoff-файлов нужен для передачи проекта DVRK Control другому ИИ-агенту.

Пакет объясняет:

```text
что такое DVRK Control;
как устроена система;
как работали через MiMo;
какая инфраструктура используется;
как устроены Tilda, YooKassa и Честный Знак;
как устроены заказы, статистика, CRM, логи и backup;
как устроен WB-модуль;
что нельзя ломать;
что делать дальше.
```

---

## 2. Полный список файлов

Полный комплект:

```text
00_INDEX_что_читать_сначала.md
01_PROJECT_OVERVIEW_DVRK_Control.md
02_WORKFLOW_как_мы_работали_через_MiMo.md
03_INFRA_SERVER_ENV_DEPLOY.md
04_TILDA_YOOKASSA_CHZ_MAIN_LOGIC.md
05_YOOKASSA_RECEIPT_PAYLOAD_AND_SAFETY.md
06_TILDA_WEBHOOK_SETTINGS.md
07_UI_UX_RULES_FOR_DVRK_Control.md
08_ORDERS_AND_CHZ_OPERATOR_WORKFLOW.md
09_STATS_CUSTOMERS_CRM.md
10_LOGS_BACKUP_SYSTEM.md
11_INTEGRATIONS_PAGE.md
12_WB_ANALYTICS_OVERVIEW.md
13_WB_MONEY_FLOW_AND_UNIT_ECONOMICS.md
14_WB_WAREHOUSES_AND_PRODUCT_LOGISTICS.md
15_WB_ADJUSTMENTS_NORMALIZATION.md
16_SECURITY_AND_DO_NOT_BREAK.md
17_CHANGELOG_v093_to_v168.md
18_NEXT_ROADMAP.md
19_MIMO_TASK_FILE_TEMPLATE.md
20_FINAL_PACKAGE_INDEX_AND_CHECKLIST.md
```

---

## 3. Порядок чтения

Читать строго по порядку.

Минимум перед любой работой:

```text
00_INDEX_что_читать_сначала.md
01_PROJECT_OVERVIEW_DVRK_Control.md
02_WORKFLOW_как_мы_работали_через_MiMo.md
16_SECURITY_AND_DO_NOT_BREAK.md
18_NEXT_ROADMAP.md
19_MIMO_TASK_FILE_TEMPLATE.md
```

Если задача про Tilda / YooKassa / ЧЗ:

```text
04_TILDA_YOOKASSA_CHZ_MAIN_LOGIC.md
05_YOOKASSA_RECEIPT_PAYLOAD_AND_SAFETY.md
06_TILDA_WEBHOOK_SETTINGS.md
08_ORDERS_AND_CHZ_OPERATOR_WORKFLOW.md
16_SECURITY_AND_DO_NOT_BREAK.md
```

Если задача про UI:

```text
07_UI_UX_RULES_FOR_DVRK_Control.md
08_ORDERS_AND_CHZ_OPERATOR_WORKFLOW.md
09_STATS_CUSTOMERS_CRM.md
10_LOGS_BACKUP_SYSTEM.md
11_INTEGRATIONS_PAGE.md
```

Если задача про WB:

```text
12_WB_ANALYTICS_OVERVIEW.md
13_WB_MONEY_FLOW_AND_UNIT_ECONOMICS.md
14_WB_WAREHOUSES_AND_PRODUCT_LOGISTICS.md
15_WB_ADJUSTMENTS_NORMALIZATION.md
16_SECURITY_AND_DO_NOT_BREAK.md
```

---

## 4. Текущая точка проекта

Текущая стабильная точка:

```text
v1.65-logs-integrations-settings
```

Состояние:

```text
/orders работает;
/chz работает;
/stats работает;
/customers работает;
/logs после v168 принят как хорошая база;
/integrations создана, но требует доработки;
/wb-модуль развит, но чистая прибыль продавца ещё не финализирована.
```

---

## 5. Самый логичный следующий шаг

Следующий логичный task:

```text
v169 — расширить /integrations
```

Цель:

```text
доработать страницу /integrations до полноценной панели YooKassa/Tilda.
```

Не трогать без необходимости:

```text
/logs;
WB formulas;
Tilda webhook parser;
YooKassa closing receipt payload;
user_ui_settings v167.
```

---

## 6. Критичные правила, которые нельзя нарушать

Главное:

```text
DVRK Control не делает первый чек.
Первый чек делает Tilda / YooKassa.
DVRK Control делает второй закрывающий чек.
ZH в SKU = нужен Честный Знак.
full_prepayment = закрывающий чек нужен.
full_payment = закрывающий чек не нужен.
unknown = проверить признак расчёта.
```

---

## 7. Secrets

Нельзя показывать, логировать или отдавать через API:

```text
YooKassa Secret Key;
Tilda webhook secret;
WB token;
Authorization headers;
.env;
passwords;
cookies.
```

В UI только masked.

Пустое secret-поле не должно затирать старый secret.

---

## 8. GET no external API

GET-страницы не должны вызывать:

```text
YooKassa API;
Tilda API;
WB API;
отправку чеков;
синхронизацию;
проверку подключения;
тяжёлые внешние операции.
```

Активные действия — только POST/action.

---

## 9. WB-критичные правила

Нельзя:

```text
считать deliveryAmount рублями;
считать returnAmount рублями;
считать все quantity продажами;
повторно вычитать эквайринг после forPay;
называть forPay чистой прибылью;
называть остаток после WB чистой прибылью;
распределять report-level deduction без отдельного режима;
удалять raw_json;
вызывать WB API на GET.
```

---

## 10. UI-критичные правила

Интерфейс должен быть:

```text
русский;
тёмный;
компактный;
центрированный;
без лишних постоянных поясняющих блоков;
с tooltip для неочевидных элементов;
с confirm для опасных действий;
с понятными disabled reasons.
```

UI-задачи без browser screenshots не принимать.

---

## 11. MiMo workflow checklist

Перед созданием task-файла:

```text
понять одну конкретную задачу;
проверить, какой vXXX следующий;
не смешивать несколько направлений;
вписать запреты;
вписать backup, если задача рискованная;
вписать browser checks;
вписать screenshots;
вписать report requirements;
вписать git commit/tag.
```

После MiMo:

```text
прочитать report_vXXX.md;
проверить screenshots;
проверить сценарии;
проверить GET no external API;
проверить secrets not exposed;
проверить git status;
только потом принять.
```

---

## 12. Минимальная проверка любой задачи

Для любой задачи:

```text
py_compile;
smoke;
страница задачи открывается;
соседние страницы открываются;
нет JS errors;
нет secrets в HTML/API/logs;
GET не вызывает внешние API;
git status понятен.
```

---

## 13. Минимальная проверка Tilda / YooKassa / ЧЗ

Проверить:

```text
/orders;
/orders/{id};
/chz;
/chz/orders/{id};

ZH + full_prepayment + нет кода → кнопка disabled;
ZH + full_prepayment + код есть → можно готовить чек;
non-ZH + full_prepayment → чек без ЧЗ;
full_payment → чек не нужен;
unknown → проверить признак расчёта;
повторная отправка succeeded/pending заблокирована.
```

---

## 14. Минимальная проверка WB

Проверить:

```text
/wb;
/wb/products;
/wb/products/{nm_id};
/wb/unit-economics;
/wb/diagnostics;
/wb/field-dictionary;
/wb/money-flow;
/wb/warehouses;
/wb/product-warehouses;

GET /wb* не вызывает WB API;
token masked;
massagor считается логично;
lopata0126 показывает корректировку до forPay;
расходы со знаком минус;
forPay не называется чистой прибылью.
```

---

## 15. Минимальная проверка /integrations

Проверить:

```text
/integrations открывается;
YooKassa block виден;
Tilda block виден;
secrets masked;
empty secret не затирает старый;
audit без secrets;
GET /integrations не вызывает внешние API;
/logs не сломан;
/orders не сломан;
/chz не сломан;
/wb не сломан.
```

---

## 16. Минимальная проверка /logs и backup

Проверить:

```text
/logs открывается;
dashboard компактный;
таблица событий работает;
backup block виден;
скачивание backup безопасно;
path traversal запрещён;
secrets не видны;
GET /logs не вызывает внешние API.
```

---

## 17. Что не делать без нового запроса пользователя

Не делать:

```text
email / Telegram уведомления;
глобальную авторизацию;
YooKassa live mode;
перевод Tilda webhook на JSON;
изменение ZH-правила;
автоматическое распределение WB report-level расходов;
удаление audit history;
перенос backup directory;
возврат внутреннего склада как основного раздела UI.
```

---

## 18. Что считать красным флагом

Если новый ИИ предлагает:

```text
сразу переписать всё приложение;
перевести Tilda webhook на JSON;
вызывать API на открытии страницы;
считать forPay чистой прибылью;
размазать deduction по товарам без методики;
убрать raw_json;
показывать tokens для удобства;
отправлять чек без confirm;
не делать screenshots;
принять PASS без проверки.
```

Это нужно остановить.

---

## 19. Как передавать пакет другому ИИ

Лучше передавать весь пакет целиком.

Порядок:

```text
сначала 00_INDEX;
потом 01–03;
потом 16;
потом нужные тематические файлы;
потом 18–20.
```

Если другой ИИ будет продолжать разработку, дать ему задачу:

```text
Сначала прочитай 00, 01, 02, 16, 18, 19, 20.
Потом предложи следующий vXXX.md, но не меняй код без моего подтверждения.
```

---

## 20. Финальное резюме

```text
DVRK Control — рабочая система, которую нельзя переписывать хаотично.
Главная схема: Tilda → YooKassa предоплата → DVRK Control закрывающий чек.
ЧЗ определяется через ZH в SKU.
WB-модуль отдельный и работает по финансовым отчётам.
forPay не чистая прибыль.
Secrets не показывать.
GET не вызывает внешние API.
Работать только пошагово через MiMo task-файлы.
Следующий логичный шаг — v169 расширить /integrations.
```
