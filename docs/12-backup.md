# 12 — Резервное копирование (Restic)

## Цель
Ежедневное инкрементальное резервное копирование критичных данных через Restic с автоматической ротацией снапшотов.

---

## Архитектура

| Компонент | Расположение |
|-----------|-------------|
| Бинарник | `/usr/bin/restic` (v0.16.4) |
| Переменные окружения (репозиторий, пароль) | `/etc/restic-env` (права `600 root`) |
| Список включаемых путей | `/etc/restic-includes.txt` |
| Список исключений | `/etc/restic-excludes.txt` |
| Скрипт предварительного дампа Postgres | `/usr/local/sbin/restic-predump.sh` (права `700 root`) |
| systemd-сервис | `/etc/systemd/system/restic-backup.service` |
| systemd-таймер | `/etc/systemd/system/restic-backup.timer` |

---

## Что бэкапится

```
/etc/letsencrypt                        — TLS-сертификаты
/etc/nginx/sites-available              — Nginx vhosts
/etc/nginx/snippets
/etc/nginx/nginx.conf
/etc/turnserver.conf                    — coturn
/etc/turnserver-dh.pem
/etc/ufw                                — UFW правила
/etc/crowdsec                           — CrowdSec конфиг
/etc/ssh/sshd_config.d                  — SSH hardening
/etc/sysctl.d                           — параметры ядра
/usr/local/etc/xray                     — Xray конфиг + ключи Reality
/var/backups/dumps/synapse-latest.dump  — Postgres дамп (Matrix)
/opt/matrix/docker-compose.yml
/opt/matrix/.env
/opt/matrix/synapse/data
/opt/matrix/element/config
/opt/livekit/docker-compose.yml
/opt/livekit/.env
/opt/livekit/config
```

Также включены сами файлы бэкапа (self-backup): systemd-юниты, скрипт предварительного дампа.

---

## Как работает

Последовательность шагов при каждом запуске (`restic-backup.service`):

1. **Pre-dump** — `restic-predump.sh` делает `pg_dump` Matrix Postgres в `/var/backups/dumps/synapse-latest.dump`
2. **Backup** — `restic backup` загружает все пути из `restic-includes.txt` в репозиторий
3. **Forget + Prune** — удаляет снапшоты сверх лимита и освобождает место:
   - `--keep-daily 7` — 7 ежедневных
   - `--keep-weekly 4` — 4 еженедельных
   - `--keep-monthly 6` — 6 ежемесячных
4. **Check** — быстрая проверка целостности репозитория

Таймер: ежедневно в **03:00** (`±30 мин` случайный сдвиг), `Persistent=true` — запустится после перезагрузки если пропустил время.

---

## Управление

```bash
# Статус таймера и время последнего запуска
systemctl list-timers restic-backup.timer

# Логи последнего запуска
journalctl -u restic-backup.service -n 50

# Список снапшотов
sudo restic --env-file /etc/restic-env snapshots

# Запустить бэкап вручную (не дожидаясь таймера)
sudo systemctl start restic-backup.service

# Проверить целостность репозитория
sudo restic --env-file /etc/restic-env check
```

---

## Восстановление

### Список файлов в снапшоте

```bash
sudo restic --env-file /etc/restic-env ls SNAPSHOT_ID /opt/matrix
```

### Восстановить конкретный файл

```bash
sudo restic --env-file /etc/restic-env restore SNAPSHOT_ID \
  --include /usr/local/etc/xray/config.json \
  --target /
```

### Восстановить Postgres (Matrix)

```bash
# 1. Извлечь дамп из снапшота
sudo restic --env-file /etc/restic-env restore SNAPSHOT_ID \
  --include /var/backups/dumps/synapse-latest.dump \
  --target /tmp/restore

# 2. Остановить Synapse
cd /opt/matrix && docker compose stop synapse

# 3. Восстановить БД
cat /tmp/restore/var/backups/dumps/synapse-latest.dump \
  | docker exec -i matrix-postgres psql -U synapse synapse

# 4. Запустить Synapse
docker compose start synapse
```

### Восстановить всё на новый сервер

```bash
sudo restic --env-file /etc/restic-env restore latest --target /
```

---

## Обновление списка путей

```bash
sudo nano /etc/restic-includes.txt
# Добавить нужные пути, затем проверить:
sudo systemctl start restic-backup.service
```

---

## Предыдущий шаг
← [10-checklist.md](10-checklist.md)
