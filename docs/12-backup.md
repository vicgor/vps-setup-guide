# 12 — Резервное копирование

## Цель
Защитить критичные данные от потери при сбое сервера или случайного удаления.

---

## Что бэкапить

| Данные | Путь | Приоритет |
|--------|------|-----------|
| Matrix Postgres (история переписки) | volume в `/opt/matrix/postgres/data/` | 🔴 Критично |
| Matrix конфиги (Synapse, Element) | `/opt/matrix/synapse/`, `/opt/matrix/element/` | 🔴 Критично |
| Xray конфиг (ключи Reality) | `/usr/local/etc/xray/config.json` | 🟠 Важно |
| Nginx vhosts | `/etc/nginx/sites-enabled/` | 🟠 Важно |
| Let's Encrypt сертификаты | `/etc/letsencrypt/` | 🟠 Важно |
| LiveKit конфиги | `/opt/livekit/` | 🟡 Средний |
| SSH конфиг | `/etc/ssh/sshd_config.d/` | 🟡 Средний |

> Postgres нельзя бэкапить простым копированием файлов — только через `pg_dump`.

---

## 1. Структура хранения

```bash
mkdir -p /opt/backups/{db,configs}
chmod 700 /opt/backups
```

---

## 2. Скрипт бэкапа

```bash
nano /opt/backups/backup.sh
```

```bash
#!/bin/bash
set -euo pipefail

BACKUP_DIR=/opt/backups
DATE=$(date +%Y%m%d-%H%M)
KEEP_DAYS=7

# --- Postgres (Matrix) ---
docker exec matrix-postgres pg_dump -U synapse synapse \
  | gzip > "$BACKUP_DIR/db/matrix-postgres-$DATE.sql.gz"

# --- Конфиги ---
tar -czf "$BACKUP_DIR/configs/configs-$DATE.tar.gz" \
  /opt/matrix/synapse \
  /opt/matrix/element \
  /opt/livekit \
  /etc/nginx/sites-enabled \
  /etc/letsencrypt \
  /etc/ssh/sshd_config.d \
  /usr/local/etc/xray/config.json \
  2>/dev/null

# --- Ротация: удалить старше KEEP_DAYS дней ---
find "$BACKUP_DIR" -name "*.gz" -mtime +$KEEP_DAYS -delete

echo "[$DATE] Backup completed"
```

```bash
chmod 700 /opt/backups/backup.sh
```

Проверка вручную:

```bash
/opt/backups/backup.sh && ls -lh /opt/backups/db/ /opt/backups/configs/
```

---

## 3. Автоматизация через systemd timer

Создать unit-файл сервиса:

```bash
nano /etc/systemd/system/backup.service
```

```ini
[Unit]
Description=VPS Backup

[Service]
Type=oneshot
ExecStart=/opt/backups/backup.sh
StandardOutput=journal
StandardError=journal
```

Создать таймер (ежедневно в 03:00):

```bash
nano /etc/systemd/system/backup.timer
```

```ini
[Unit]
Description=Daily VPS Backup

[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
systemctl daemon-reload
systemctl enable --now backup.timer

# Проверить
systemctl list-timers backup.timer
```

---

## 4. Удалённое хранение (опционально)

Добавить в конец `backup.sh` копирование на внешний хост через rsync:

```bash
# rsync на удалённый сервер
rsync -az /opt/backups/ backup-user@REMOTE_HOST:/backups/vps/
```

Или через rclone в облако (S3, Backblaze B2, Google Drive):

```bash
apt install rclone -y
rclone config  # настроить хранилище
# Добавить в backup.sh:
rclone sync /opt/backups/ remote:vps-backups/
```

---

## 5. Восстановление Postgres

```bash
# Остановить Synapse (не трогать postgres-контейнер)
cd /opt/matrix && docker compose stop synapse

# Восстановить из дампа
gunzip -c /opt/backups/db/matrix-postgres-YYYYMMDD-HHMM.sql.gz \
  | docker exec -i matrix-postgres psql -U synapse synapse

# Запустить Synapse
docker compose start synapse
```

---

## Следующий шаг
← [10-checklist.md](10-checklist.md)
