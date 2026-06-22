# Улучшения безопасности сервера

Список задач по результатам аудита от 2026-05-02. Все пункты выполнены по состоянию на 2026-06-22.

---

## ✅ Выполнено

### 1. Права доступа к конфигурации Xray

`/usr/local/etc/xray/config.json` — права изменены на `640 root:nogroup`, три файла бэкапов удалены.

### 2. Netdata

Доступ ограничен через UFW: разрешён только с IP оператора (`91.149.143.96`), остальным — запрещён.

### 3. Docker daemon.json

Создан `/etc/docker/daemon.json` с ограничением логов, `live-restore`, `no-new-privileges`.

### 4. SSH sshd_config

Старый `hardening.conf` отключён. Активен `99-hardening.conf` с полным набором ограничений (`Port 2222`, `AllowUsers deploy`, `PasswordAuthentication no` и др.).

---

## ✅ Что настроено корректно (на момент аудита)

| Параметр | Статус |
|----------|--------|
| SSH: порт 2222, только ключи, только пользователь `deploy` | ✅ |
| CrowdSec + firewall bouncer | ✅ |
| UFW активен | ✅ |
| unattended-upgrades | ✅ |
| AppArmor | ✅ |
| sysctl: tcp_syncookies, rp_filter, no redirects | ✅ |
| Docker: все порты через `127.0.0.1` | ✅ |
| `.env` файлы: права `600` | ✅ |
| Xray: запуск от `nobody`, `NoNewPrivileges=true` | ✅ |
| Xray: блокировка приватных IP, BitTorrent | ✅ |
