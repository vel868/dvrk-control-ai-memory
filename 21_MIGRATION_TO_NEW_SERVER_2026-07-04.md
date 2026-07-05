# 21 — Migration to new production server, 2026-07-04

## Result

Project DVRK Control was migrated from the old server to the new server.

New working server:

```text
194.87.80.116
```

Working domain:

```text
https://controldev.bussanmaster.ru
```

Old server remains as temporary reserve:

```text
195.133.31.23
```

## What was done

1. New Ubuntu 24.04 server was prepared.
2. PostgreSQL, Nginx, Certbot, Python, venv, Node/npm and required packages were installed.
3. Fresh project archive was created on the old server through MiMo.
4. Fresh project archive was copied to the new server.
5. Project was restored to:

```text
/opt/dvrk_control_app
```

6. Main PostgreSQL database was restored:

```text
dvrk_control_dev
```

7. Separate WB database was restored:

```text
dvrk_wb_dev
```

8. Service was restored and is running:

```text
dvrk-control-dev
```

9. Application listens locally on:

```text
127.0.0.1:8010
```

10. Nginx reverse proxy was configured.
11. Basic Auth was enabled through Nginx.
12. SSL was issued with Certbot.
13. New domain was configured:

```text
controldev.bussanmaster.ru
```

14. MiMo was installed on the new server and confirmed project structure.
15. Manual backup script was created:

```bash
bash /opt/dvrk_control_app/scripts/manual_full_backup.sh
```

## Final checks

The following internal routes returned 200:

```text
/orders
/wb
/integrations
/logs
```

External HTTPS without password returns 401, which is correct:

```text
https://controldev.bussanmaster.ru/orders -> 401 Unauthorized
```

Open ports:

```text
80  — nginx
443 — nginx
8010 — uvicorn local
```

## Databases after migration

Databases on new server:

```text
dvrk_control
dvrk_control_dev
dvrk_wb_dev
postgres
```

Important active databases:

```text
dvrk_control_dev
dvrk_wb_dev
```

WB verification after restore:

```text
wb_sales_report_rows: about 15455 rows
wb_settings: 2 rows
```

## Backup after migration

Auto daily DVRK backup was not found. User wants manual backup only.

Manual backup command:

```bash
bash /opt/dvrk_control_app/scripts/manual_full_backup.sh
```

Backup path:

```text
/opt/dvrk_backups/daily/dvrk_full_backup_YYYY-MM-DD_HH-MM-SS.tar.gz
```

Backup can be downloaded from web UI:

```text
Логи → Скачать backup
```

## Do not upload to public GitHub

Do not upload `.env`, database dumps, backup archives, htpasswd, SSL keys, tokens, passwords, buyer data, or raw logs with personal data.
