# DVRK Control — AI Memory

Обновлено: 2026-07-04 после миграции на новый сервер.

Это публичная AI-память проекта DVRK Control. Файл не должен содержать пароли, токены, `.env`, дампы баз, ключи YooKassa, WB API token, Tilda token, персональные данные покупателей или SSL-ключи.

## 1. Текущее состояние

DVRK Control — FastAPI-приложение для управления заказами Tilda, YooKassa, Честным Знаком, складом, аудитом, WB-аналитикой, логами и backup.

Текущий рабочий сервер:

```text
194.87.80.116
```

Рабочий домен:

```text
https://controldev.bussanmaster.ru
```

Проект на сервере:

```text
/opt/dvrk_control_app
```

Сервис приложения:

```text
dvrk-control-dev
```

Приложение работает локально на:

```text
127.0.0.1:8010
```

Nginx проксирует внешний HTTPS на приложение.

Состояние после миграции:

```text
/orders — работает
/wb — работает
/integrations — работает
/logs — работает
SSL — работает
Basic Auth через nginx — работает
HTTPS без логина/пароля возвращает 401 Unauthorized — это правильно
```

## 2. Серверы

Новый сервер:

```text
IP: 194.87.80.116
OS: Ubuntu 24.04
Статус: рабочий сервер
```

Старый сервер:

```text
IP: 195.133.31.23
Статус: резерв после миграции
```

Старый сервер держать несколько дней после успешной миграции, затем отключать только после проверки сайта, WB, backup, Tilda webhook и YooKassa.

## 3. Базы данных

Основная база:

```text
dvrk_control_dev
```

Важные схемы:

```text
agents
audit
catalog
chz
finance
inventory
manual
public
sales
system_core
tax
tilda
```

WB база:

```text
dvrk_wb_dev
```

Таблицы WB:

```text
wb_adjustments
wb_api_request_logs
wb_field_dictionary
wb_sales_report_rows
wb_sales_reports
wb_settings
wb_sync_runs
```

На момент переноса WB база восстановлена, `/wb` работает, строк в `wb_sales_report_rows` было около 15455.

Роли PostgreSQL:

```text
dvrk_app
dvrk_user
```

Пароли не хранить в GitHub.

## 4. Tilda / YooKassa / Честный Знак

Критичная логика:

1. Tilda принимает оплату через YooKassa.
2. Tilda формирует первый чек предоплаты.
3. DVRK Control делает только второй/закрывающий чек.
4. Закрывающий чек делается после оплаты и после ввода кода Честного Знака или выбора “ЧЗ не нужен”.
5. `payment_mode=full_prepayment` значит закрывающий чек нужен.
6. `payment_mode=full_payment` значит второй чек не нужен.
7. Если признак расчёта неизвестен, чек нельзя отправлять автоматически.
8. `ZH` в SKU означает товар с Честным Знаком.
9. Код ЧЗ не кодировать Base64.
10. Не отправлять чек повторно, если чек уже `succeeded` или `pending`.
11. Не отправлять чек на GET-запрос.

Важные поля Tilda webhook:

```text
payment[orderid]
payment[systranid]
payment[amount]
payment[subtotal]
payment[delivery_price]
payment[delivery_fio]
Email
Phone
payment[products][i][sku]
payment[products][i][name]
payment[products][i][quantity]
payment[products][i][price]
payment[products][i][amount]
```

Стабильные настройки Tilda webhook:

```text
API METHOD: Off
Передавать данные по товарам массивом: включено
Отправлять только после оплаты: включено
application/json: выключено
Cookies: выключены
externalid: выключен
```

## 5. WB логика

WB модуль отдельный от Tilda/YooKassa.

```text
WB данные хранятся в отдельной базе dvrk_wb_dev
GET /wb не должен вызывать WB API
WB API нельзя запускать без явной команды пользователя
```

Правила расчётов:

```text
forPay — начисление WB по товарной строке, не чистая прибыль
bankPaymentSum — сумма на уровне отчёта, не точная сумма по конкретному товару
deliveryAmount и returnAmount — не рубли
deliveryService и rebillLogisticCost — денежные расходы
расходы показывать с минусом
report-level deduction нельзя распределять без отдельной методики
```

## 6. Backup

Автоматический ежедневный DVRK backup на новом сервере не найден. Пользователь хочет запускать backup вручную.

Ручной backup-скрипт:

```bash
bash /opt/dvrk_control_app/scripts/manual_full_backup.sh
```

Backup сохраняется в:

```text
/opt/dvrk_backups/daily/dvrk_full_backup_YYYY-MM-DD_HH-MM-SS.tar.gz
```

Backup можно скачать через сайт:

```text
Логи → Скачать backup
```

Backup содержит проект, `.env`, `.mimocode`, `AGENTS.md`, основную базу `dvrk_control_dev`, WB базу `dvrk_wb_dev`, nginx config, systemd service, htpasswd и отчёт.

Не хранить backup-архивы в GitHub.

## 7. Nginx / SSL / Auth

Рабочий домен:

```text
https://controldev.bussanmaster.ru
```

SSL выпущен через Certbot. Basic Auth включён через Nginx. Логин/пароль не хранить в GitHub.

Внешний HTTPS без пароля должен возвращать:

```text
401 Unauthorized
```

Webhook/health endpoints при необходимости должны быть доступны без Basic Auth, чтобы внешние сервисы могли отправлять события.

## 8. MiMo

MiMo установлен на новом сервере и видит проект:

```text
/opt/dvrk_control_app
```

MiMo подтвердил структуру проекта DVRK Control.

## 9. UI правила

Пользователь предпочитает тёмную тему, русский интерфейс, визуально центрированные страницы, тонкие тёмные scrollbar, подсказки через tooltip, confirm для опасных действий, не засорять интерфейс постоянными поясняющими блоками.

## 10. Открытые задачи

```text
проверить скачивание backup через сайт
держать старый сервер резервом несколько дней
проверить Tilda webhook после окончательного переключения
проверить YooKassa test/prod режим перед боевым запуском
позже можно сделать нормальную авторизацию внутри приложения вместо nginx Basic Auth
позже можно скрыть ключи на странице подключения, если пользователь решит
```

## 11. Как использовать эту память

Когда нужно продолжить работу с ChatGPT:

```text
Посмотри память проекта: https://github.com/vel868/dvrk-control-ai-memory
```

ChatGPT может читать публичный репозиторий по ссылке, но не может сам записывать изменения в GitHub. Если нужно обновить память, ChatGPT готовит текст, а пользователь вручную вставляет его в GitHub.
