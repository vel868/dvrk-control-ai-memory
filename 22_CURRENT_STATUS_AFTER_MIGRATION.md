# 22 — Current status after migration

## Working URLs

```text
https://controldev.bussanmaster.ru
```

## Status

```text
SSL: OK
Basic Auth: OK
/orders: OK
/wb: OK
/integrations: OK
/logs: OK
Manual backup: OK
MiMo on new server: OK
```

## Server

```text
IP: 194.87.80.116
Project: /opt/dvrk_control_app
Service: dvrk-control-dev
Local app: 127.0.0.1:8010
```

## Databases

```text
dvrk_control_dev — main DB
dvrk_wb_dev — WB DB
```

## Manual backup

```bash
bash /opt/dvrk_control_app/scripts/manual_full_backup.sh
```

Backup appears in:

```text
/opt/dvrk_backups/daily/
```

Web download:

```text
Логи → Скачать backup
```

## Next actions

- Keep old server `195.133.31.23` for a few days.
- Verify real Tilda webhook after final switch.
- Verify YooKassa test/prod mode before live receipts.
- Use GitHub memory repo for AI context only, without secrets.
