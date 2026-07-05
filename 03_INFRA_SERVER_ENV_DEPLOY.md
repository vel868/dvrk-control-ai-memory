# 03_INFRA_SERVER_ENV_DEPLOY — инфраструктура, сервер, окружение и деплой

Этот файл описывает техническую среду проекта DVRK Control: где находится приложение, как оно запускается, как работали с VPS, какие проверки использовались, как делались backup и какие правила нельзя нарушать при деплое.

Файл предназначен для другого ИИ-агента, который будет помогать пользователю продолжать проект.

---

# 1. Общая техническая картина

DVRK Control — это серверное FastAPI-приложение, развернутое на VPS.

Основная рабочая директория проекта:

```bash
/opt/dvrk_control_app
```

Основной dev-сервис systemd:

```bash
dvrk-control-dev
```

Dev-домен:

```text
https://control-dev.bussanmaster.ru
```

Основная ветка git:

```text
master
```

Проект развивался итерациями через task-файлы вида:

```text
/opt/dvrk_control_app/vXXX.md
```

После выполнения MiMo создавал отчёты вида:

```text
/opt/dvrk_control_app/report_vXXX.md
```

Текущий важный последний tag на момент передачи:

```text
v1.65-logs-integrations-settings
```

---

# 2. Как начинать любую новую работу

Перед изменениями другой ИИ или MiMo должен всегда начинать с проверки состояния проекта:

```bash
cd /opt/dvrk_control_app
git status
```

Нельзя начинать задачу, если:

```text
есть незакоммиченные изменения, которые непонятно откуда взялись;
есть незавершённый task-файл старой версии;
есть report предыдущей задачи, который пользователь ещё не принял;
непонятно, какой текущий tag;
непонятно, в какой БД/среде выполняется задача.
```

Если есть сомнения, нужно сначала остановиться и объяснить пользователю, что найдено.

---

# 3. Типовой формат выполнения задачи

Обычно работа шла так:

```bash
cd /opt/dvrk_control_app
source .venv/bin/activate
python -m py_compile app/main.py app/routes/*.py app/core/*.py app/integrations/*.py app/audit/*.py scripts/*.py
systemctl restart dvrk-control-dev
./scripts/smoke_test.sh
python scripts/check_..._vXXX.py
journalctl -u dvrk-control-dev -n 120 --no-pager
```

Не нужно запускать все старые проверки подряд. В task-файлах обычно явно писалось:

```text
Не запускать старые проверки v127-vXXX без причины.
```

Это важно, потому что старые проверки могут быть устаревшими, долгими или не соответствовать текущей логике.

---

# 4. Systemd service

Приложение запускается через systemd-сервис:

```bash
systemctl restart dvrk-control-dev
systemctl status dvrk-control-dev --no-pager
journalctl -u dvrk-control-dev -n 120 --no-pager
```

После любых изменений нужно проверять:

```text
сервис запустился;
нет traceback;
нет критических ошибок в journalctl;
страницы открываются;
GET-страницы не вызывают внешние API.
```

Если сервис не поднялся, нельзя продолжать делать новые изменения. Сначала нужно вернуть приложение в рабочее состояние.

---

# 5. Python virtual environment

Перед запуском скриптов обычно активировалось окружение:

```bash
cd /opt/dvrk_control_app
source .venv/bin/activate
```

После этого запускались compile/check/debug/browser scripts.

---

# 6. Backup перед изменениями

Перед серьёзными задачами делался backup базы.

Типовой шаблон:

```bash
mkdir -p /opt/backups
pg_dump -h 127.0.0.1 -U dvrk_app dvrk_control_dev > /opt/backups/pre_vXXX_<task_name>_$(date +%Y%m%d_%H%M%S).sql
```

Для WB-задач иногда использовалась отдельная БД:

```bash
pg_dump -h 127.0.0.1 -U dvrk_app dvrk_wb_dev > /opt/backups/pre_vXXX_wb_<task_name>_$(date +%Y%m%d_%H%M%S).sql
```

Важное правило:

```text
Перед задачей нужно понять, какая БД затрагивается.
Если задача про основную систему — dvrk_control_dev.
Если задача про WB-модуль — могла использоваться dvrk_wb_dev.
Если не уверен — сначала проверить текущие настройки проекта.
```

---

# 7. Git workflow

После успешной задачи обычно делался commit и tag.

Пример:

```bash
git status
git add .
git commit -m "v1.65 redesign logs and add integrations settings"
git tag v1.65-logs-integrations-settings
```

Правило:

```text
Не делать commit/tag, если проверки не прошли.
Не делать tag, если пользователь ещё не видел report и задача сомнительная.
Не смешивать несколько крупных задач в один commit.
```

История версий важна, потому что пользователь и ИИ часто опирались на теги:

```text
v1.47-wb-field-dictionary
v1.48-wb-period-buttons-ui
v1.49-wb-period-filter-fix
v1.50-wb-june-normalization-fix
v1.51-wb-expenses-dates-reconcile
v1.52-wb-real-sales-classification
v1.53-wb-money-flow-forpay-payout
v1.54-wb-commission-nonbuyout-logic
v1.55-wb-nonbuyout-reconcile-diff
v1.56-wb-forpay-bankpayment-semantics
v1.57-wb-logistics-cost-columns
v1.58-wb-warehouse-logistics
v1.59-wb-negative-expense-display
v1.60-wb-product-warehouse-logistics
v1.61-wb-forpay-adjustment-reconciliation
v1.62-wb-hidden-adjustments-audit
v1.63-wb-adjustments-normalization
v1.64-stats-customers-ui-settings
v1.65-logs-integrations-settings
```

Некоторые номера task-файлов и tag-версий не совпадали один-в-один. Это нормально. Важнее смотреть report и git tag.

---

# 8. Smoke tests

В проекте использовался общий smoke-test:

```bash
./scripts/smoke_test.sh
```

Он проверял базовую доступность ключевых страниц.

После smoke-test часто запускались специальные проверки задачи:

```bash
python scripts/check_<feature>_vXXX.py
```

Для UI-задач дополнительно создавались Playwright/browser-проверки:

```bash
python scripts/check_<feature>_browser_vXXX.py
```

Обычно task-файл требовал screenshots в `/tmp`, например:

```text
/tmp/v168_logs_dashboard.png
/tmp/v168_integrations_top.png
```

Другой ИИ должен требовать реальные browser/DOM-проверки для UI, а не только `py_compile`.

---

# 9. Проверки через browser / DOM

Для UI задач важно не принимать отчёт только по словам MiMo.

Нужно проверять:

```text
страница реально открывается;
нужные элементы есть в DOM;
таблица не расползлась;
кнопки видны;
секреты не отображаются;
нет JS errors;
GET-страница не вызывает внешние API;
screenshots сохранены.
```

Примеры UI-задач, где browser-проверка была обязательна:

```text
/logs redesign;
/integrations page;
/stats UI settings;
/customers UI settings;
/wb product warehouse pages;
/wb money flow pages.
```

---

# 10. Внешние API: критичное правило GET

Очень важное архитектурное правило:

```text
GET-страницы не должны вызывать внешние API.
```

Это касается:

```text
YooKassa API;
Tilda API;
WB API.
```

GET-страница может читать БД и показывать текущие данные.

Внешний API можно вызывать только по явному действию оператора, обычно через POST:

```text
POST /api/integrations/yookassa/check
POST /api/integrations/tilda/check
ручная WB sync
ручная отправка закрывающего чека
```

Почему это важно:

```text
страница может обновляться автоматически;
оператор может просто открыть вкладку;
поисковые/проверочные скрипты могут ходить по GET;
нельзя случайно отправить чек или начать синхронизацию;
нельзя спамить WB API.
```

---

# 11. Основные страницы системы

На момент передачи есть такие важные страницы:

```text
/orders
/orders/{id}
/chz
/chz/orders/{id}
/stats
/customers
/logs
/system
/agents
/tilda
/integrations
/wb
/wb/sync
/wb/settings
/wb/reports
/wb/reports/{id}
/wb/products
/wb/products/{nm_id}
/wb/unit-economics
/wb/logs
/wb/diagnostics
/wb/field-dictionary
/wb/calculation-rules
/wb/money-flow
/wb/warehouses
/wb/warehouses/{warehouse}
/wb/product-warehouses
```

Новые задачи не должны ломать старые страницы.

---

# 12. Основные запреты безопасности

Нельзя показывать на страницах:

```text
.env;
WB token;
YooKassa Secret Key;
Tilda secret/token;
API keys;
passwords;
authorization headers;
cookies.
```

Нельзя логировать:

```text
secret key;
token;
password;
полные credential values;
полный .env.
```

На странице `/integrations` секреты должны быть замаскированы:

```text
••••••••
```

Если пользователь сохраняет пустой secret, старый secret не должен затираться.

---

# 13. Базы и таблицы, которые появились в ходе работы

Точные имена таблиц нужно проверять по миграциям и коду, но по истории проекта важны:

```text
user_ui_settings
integration_settings
integration_settings_audit
wb_adjustments
```

`user_ui_settings` появилась в v167 для сохранения UI-настроек только на:

```text
/stats
/customers
```

`integration_settings` и `integration_settings_audit` появились в v168 для страницы:

```text
/integrations
```

`wb_adjustments` появилась в v165 для нормализации корректировок WB.

---

# 14. UI settings v167

В v167 была добавлена persist-логика только для двух страниц:

```text
/stats
/customers
```

Важно:

```text
не подключать UI settings ко всему сайту без отдельного решения;
/orders, /chz, /wb, /logs, /agents не должны быть затронуты;
URL-параметры имеют приоритет над сохранёнными настройками.
```

Текущий tag:

```text
v1.64-stats-customers-ui-settings
```

---

# 15. Logs + Integrations v168

В v168:

```text
/logs был переделан в компактный dashboard;
создана страница /integrations;
на /integrations вынесены настройки YooKassa и Tilda;
секреты маскируются;
настройки хранятся в integration_settings;
история — в integration_settings_audit.
```

Текущий tag:

```text
v1.65-logs-integrations-settings
```

Что осталось доработать:

```text
/integrations пока рабочая база, но её можно расширять;
нужно сделать более полноценные блоки YooKassa/Tilda;
не делать email/Telegram уведомления, пользователь решил пока не делать.
```

---

# 16. Backup проекта и Windows pull

В проекте была создана страница `/logs`, где отображался backup.

Backup-архивы лежат примерно в:

```text
/opt/dvrk_backups/daily/
```

Пример файла:

```text
dvrk_full_backup_2026-07-02_17-18-25.tar.gz
```

На Windows у пользователя использовался pull backup:

```text
C:\DVRK_BACKUPS
```

SSH key:

```text
C:\Users\mrdob\.ssh\dvrk_backup_ed25519
```

Известная проблема:

```text
кириллица в PowerShell логах могла отображаться некрасиво;
но сам backup работал.
```

---

# 17. Как писать новый task-файл

Правильный task-файл должен содержать:

```text
# vXXX — название задачи

Рабочая папка
Цель
Запреты
Перед началом
Backup
Задачи по файлам
Backend
Frontend
Диагностика
Browser/DOM проверки
Технические проверки
Документация
Отчёт
Git commit/tag
Удаление task-файла
Финальный ответ
```

В task-файле нужно быть конкретным:

```text
какие routes создать;
какие endpoints;
какие таблицы;
какие поля;
какие кнопки;
какие страницы не трогать;
какие проверки запустить;
что написать в report_vXXX.md.
```

Пользователь предпочитает, чтобы ему дали один bash-блок, который создаёт task-файл, и короткую команду для MiMo:

```text
Прочитай vXXX.md и выполни автономно.
```

---

# 18. Как проверять report от MiMo

Нельзя принимать report автоматически.

Нужно проверить:

```text
что задача сделана именно в нужном scope;
что не затронуты запрещённые страницы;
что проверки не фиктивные;
что screenshots есть;
что secret не раскрыт;
что GET не вызвал внешние API;
что старые критичные сценарии не сломаны;
что арифметика/бизнес-логика сходится.
```

Если report говорит PASS, но скрин показывает проблему — задача не принимается полностью.

Так уже было много раз в WB-модуле: PASS был, но логика или UI требовали уточнения.

---

# 19. Что не делать без отдельного решения пользователя

Пользователь отдельно решил:

```text
не делать email/Telegram уведомления сейчас;
WB пока остановить как достаточно нормальный;
не расширять UI settings на весь сайт;
не запускать сразу большие переделки без обсуждения.
```

Перед такими задачами нужно сначала обсуждать.

---

# 20. Краткая текущая точка проекта

На момент создания этого handoff-пакета:

```text
Tilda/YooKassa/ЧЗ логика работает;
второй закрывающий чек настроен;
/orders и /chz рабочие;
/stats и /customers рабочие;
на /stats и /customers сохраняются UI-настройки;
/logs стал компактным dashboard;
/integrations создана как страница подключений;
WB-модуль имеет продвинутую аналитику по forPay, логистике, складам и корректировкам;
backup и логи есть;
следующие задачи — улучшение /integrations или операторского workflow /orders + /chz.
```

